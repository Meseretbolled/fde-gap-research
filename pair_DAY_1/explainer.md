# Day 1 — Explainer

**Question answered:** Does position bias in LLM-as-judge affect a single-response
rubric judge the same way it affects pairwise judges, and how do I detect it
in my held-out eval data?
**Written for:** Meseret Bolled
**Date:** 2026-05-04

---

## Introduction

Meseret's `TONE_JUDGE_PROMPT` in `scoring_evaluator.py` presents 5 tone
markers in a fixed order and asks Qwen3 for a binary 0/1 on each. Her
`methodology_rationale.md` defends the judge by citing cross-family evaluation
(different model from the agent). That defence is correct for **self-preference
bias** — but it says nothing about **position bias**, which is a completely
separate failure mode. The question is whether the fixed order
`direct → grounded → honest → professional → non_condescending` systematically
inflates the first criteria over the last, and whether that distortion touches
the +25.4% Delta A lift figure.

---

## The Load-Bearing Mechanism

Position bias in LLM judges comes from how autoregressive models generate
tokens. Each output token is conditioned on all prior tokens. When a judge
scores criterion 1 (`direct`), it has only the prompt as context. When it
scores criterion 5 (`non_condescending`), it has already generated four
scores — and those prior scores act as an implicit anchor. Two effects follow:

**Primacy effect:** Criteria listed first receive more "fresh" attention from
the model. The prompt tokens for criterion 1 are closer to the start of the
context and carry higher attention weights in early layers than criterion 5,
which is buried deeper.

**Consistency pressure:** After the model emits `"direct": 1`, it is under
implicit pressure to be consistent. A generous first score raises the baseline
for subsequent scores — not because the email got better, but because the
model's prior output now shapes its next output.

**Key distinction from pairwise bias:** In pairwise judges (A vs B), position
bias means the judge favours whichever response appears first in the prompt.
In a rubric judge like Meseret's, the bias is **inter-criteria**: criterion
order within a single prompt affects relative pass rates across criteria —
not which response wins.

This means using a different model family (Qwen3 vs the agent's backbone)
does **not** mitigate position bias. Cross-family evaluation prevents the judge
from preferring outputs that stylistically match its own training distribution.
It does nothing about the order in which criteria are presented.

---

## Show It

The detection test is simple: re-run the tone judge on the same 52 held-out
outputs with the criteria order **reversed** (`non_condescending → professional
→ honest → grounded → direct`) and compare per-criterion pass rates.

```python
import json
from pathlib import Path
from collections import defaultdict

# Load held-out results (assume each result has per-criterion tone scores)
results_path = Path("results/ablation_harness_report.json")
data = json.loads(results_path.read_text())

# Simulate: original order vs reversed order pass rates
# In practice, re-run scoring_evaluator.py with reversed TONE_MARKERS list

TONE_MARKERS_ORIGINAL = ["direct", "grounded", "honest", "professional", "non_condescending"]
TONE_MARKERS_REVERSED = list(reversed(TONE_MARKERS_ORIGINAL))

# If you have per-criterion scores saved, compare pass rates:
def pass_rate_by_criterion(results, marker_order):
    counts = defaultdict(lambda: {"pass": 0, "total": 0})
    for r in results:
        for dim in r.get("dimensions", []):
            if dim["dimension"] == "tone_compliance":
                evidence = dim.get("evidence", "")
                for marker in marker_order:
                    # parse "direct:1" style evidence string
                    if f"{marker}:1" in evidence:
                        counts[marker]["pass"] += 1
                    counts[marker]["total"] += 1
    return {k: v["pass"] / v["total"] for k, v in counts.items() if v["total"] > 0}

# Compare: if direct pass rate drops when listed last, position bias is present
original_rates = pass_rate_by_criterion(data["detailed_logs"]["delta_sft_trained"],
                                         TONE_MARKERS_ORIGINAL)
print("Original order pass rates:", original_rates)

# Red flag threshold: > 8 percentage point gap between first and last criterion
# on the same set of emails indicates position bias is material
first = list(original_rates.values())[0]
last  = list(original_rates.values())[-1]
print(f"First-to-last gap: {abs(first - last):.2%}")
print("Position bias likely" if abs(first - last) > 0.08 else "Position bias not detected")
```

If the gap between `direct` (first) and `non_condescending` (last) is
consistently > 8 pp across runs, rotate the criteria order and average the two
runs. If the gap is < 5 pp, disclose it as a known limitation and move on —
the effect is below the noise floor of a 52-task evaluation.

---

## Adjacent Concepts

**Length bias** is the other major rubric-judge failure mode. Longer outputs
score higher on `grounded` and `professional` simply because more words give
the model more signal to latch onto. Check if Meseret's high-scoring emails
are systematically longer than low-scoring ones — a scatter plot of word count
vs tone score will reveal this in one minute.

**Self-preference bias** is what the cross-family defence actually addresses:
a Qwen3 judge would give inflated tone scores to outputs that sound like Qwen3.
Using a different family breaks this. Meseret's methodology is correct on this
point.

**Calibration vs discrimination:** Even a biased judge can be useful if the
bias is *consistent across conditions*. If position bias inflates `direct`
equally for the base model and the trained model, the *relative* lift
(+25.4%) is unaffected even though the absolute scores are inflated. The
threat is only if the trained model's outputs happen to front-load the criteria
that benefit from primacy — which is worth a one-off audit.

---

## Practical FDE Rule

1. **Always check criterion pass rate variance** before reporting rubric judge
   results. A healthy rubric has < 10 pp spread across criterion pass rates on
   the same document set.
2. **Rotate and average** when position bias is detected: run the judge twice
   with reversed criterion order and average the scores. Two API calls per
   document, bias mostly cancelled.
3. **Disclose, don't hide.** If you ship without rotation, add one sentence to
   the methodology: "Criteria order is fixed; position bias has not been
   audited and may inflate pass rates for criteria listed first."

---

## Sources

- Zheng et al. (2023), "Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena" — Section 4.2 covers position bias in pairwise judges; the inter-criteria analogue follows directly from the same attention-weight argument.
- Wang et al. (2023), "Large Language Models are not Fair Evaluators" — demonstrates primacy and recency effects in rubric-style evaluation and proposes the rotation-and-average mitigation.
