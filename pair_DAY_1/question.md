# Day 1 — My Question

**Asker:** Meseret Bolled
**Explainer:** Gashaw Bekele
**Topic area:** Evaluation and Statistics — LLM-as-judge biases
**Date:** 2026-05-04

---

## The Question

In `tenacious-bench/src/evaluation/scoring_evaluator.py` (lines 66–82),
my `TONE_JUDGE_PROMPT` presents 5 tone markers in a **fixed order** and
asks Qwen3 to return a binary 0 or 1 for each:

```
1. direct
2. grounded
3. honest
4. professional
5. non_condescending
```

My `methodology_rationale.md` defends the judge with one argument:
using a **different model family** from the agent being evaluated
prevents self-preference leakage. That defence is cited as correct.

**My specific gap:**
I do not know whether **position bias** — the tendency of an LLM judge
to inflate scores for criteria listed earlier in the prompt — applies
to a single-response rubric judge the same way it applies to pairwise
comparison judges. In pairwise evaluation, position bias means the judge
favours whichever response appears first. But in my setup there is only
one response, and the judge is scoring 5 separate criteria sequentially.

The mechanism I cannot explain: does the order in which I list criteria
(direct first, non_condescending last) cause the judge to systematically
give higher pass rates to `direct` and `grounded` than to `non_condescending`
— not because the emails are actually better on those criteria, but because
of how the judge's autoregressive generation works?

And if that bias exists: how would I detect it using my existing
52 held-out task results?

---

## What I Already Know

- **Cross-family defence is correct but incomplete.** Using Qwen3 to judge
  a Qwen2.5-based agent prevents the judge from preferring outputs that
  stylistically match its own training. This addresses self-preference bias.
  It says nothing about position bias.

- **The prompt is fixed-order.** My `TONE_JUDGE_PROMPT` always presents
  criteria in the same sequence. I have never rotated the order or tested
  whether reversing it changes the scores.

- **Tone compliance contributes 15% of the total score** (weight 0.15 in
  `DIMENSION_WEIGHTS`). My dashboard shows a +25.4% relative improvement
  (0.751 → 0.941 mean score) across 52 held-out tasks. If position bias
  is inflating the first two tone criteria, some of that lift is an
  artifact of prompt ordering rather than genuine model improvement.

- **I use `passed = total >= 4`** (line 368), meaning an email needs 4
  out of 5 tone criteria to pass. A bias that systematically inflates
  the first two criteria could push borderline emails over that threshold.

---

## Why This Gap Matters for FDE Work

This is not only my situation. Two common FDE evaluation scenarios depend
on understanding this:

1. **Reporting benchmark results to a client.** Every time I present a
   rubric-judge score as evidence of model improvement, I am implicitly
   claiming the judge is unbiased. If position bias is present and I have
   not audited for it, I am overstating confidence in the result. A client
   engineer who asks "how do you know the judge isn't inflating early
   criteria?" deserves a real answer.

2. **Designing rubric prompts for new benchmarks.** If I build another
   evaluation benchmark for a different client engagement, I will face the
   same design choice: what order do I list criteria? Without understanding
   position bias mechanics I will make that choice arbitrarily, and my
   benchmark will have an unexamined systematic error baked in.

---

## Explainer Scope

Please cover:

1. The mechanism of position bias in a **single-response rubric judge**
   specifically — not pairwise. How does autoregressive generation cause
   criteria listed first to receive different treatment than criteria listed
   last? Name the attention or token-generation dynamic that drives it.

2. Whether my cross-family defence (`methodology_rationale.md`) addresses
   position bias or only self-preference bias — and why the two are
   distinct failure modes.

3. A minimal code example or audit procedure I can run on my existing
   52 held-out results to detect whether position bias is present in
   my `TONE_JUDGE_PROMPT`. What would a positive signal look like in the
   data? What threshold distinguishes "material bias" from "noise"?

4. The practical fix and disclosure rule: when should I rotate criteria
   order and re-run, and when is it sufficient to disclose position bias
   as a named limitation without re-running?

**Out of scope:** pairwise judge position bias (A vs B ordering), length
bias, self-preference bias, multi-model judge ensembles — focus only on
criterion-order effects in a single-response rubric judge.
