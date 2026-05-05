# Day 1 — My Question

**Asker:** Meseret Bolled
**Explainer:** Gashaw Bekele
**Topic area:** Evaluation and Statistics — LLM-as-judge biases
**Date:** 2026-05-04

---

## The Question

In my Week 11 benchmark, I used an LLM judge to score sales emails on
5 tone criteria presented in a fixed order. The judge returns a binary
pass or fail for each criterion, and an email must pass 4 out of 5
to be considered tone-compliant.

My evaluation methodology defends the judge on one ground: using a
different model family from the agent being evaluated prevents the judge
from favouring outputs that stylistically match its own training.

**The gap I cannot close:**
I do not know whether **position bias** — the tendency of an LLM judge
to inflate scores for criteria presented earlier in the prompt — applies
to a single-response rubric judge the same way it applies to pairwise
judges. In pairwise evaluation, position bias means the judge favours
whichever response appears first. My setup is different: one response,
five criteria scored sequentially by the same model in one pass.

Does the fixed order in which I present the five criteria cause the judge
to systematically give higher pass rates to the criteria listed first —
not because the emails are genuinely better on those criteria, but because
of how the model generates its scores one token at a time?

And if that bias exists: how would I detect it using the held-out
evaluation results I already have?

---

## What I Already Know

- **The cross-family defence addresses a different problem.** It prevents
  the judge from preferring outputs that sound like itself. It says nothing
  about whether criteria listed first in a prompt are scored more generously
  than criteria listed last.

- **The prompt order has never been tested.** I have always presented the
  criteria in the same fixed sequence. I have never rotated the order or
  checked whether reversing it changes the scores.

- **The tone dimension contributes 15% of the total benchmark score.**
  My benchmark reports a +25.4% relative improvement for the trained model
  over the baseline across 52 held-out tasks. If position bias is inflating
  the first two criteria, some of that lift is an artifact of prompt
  ordering rather than genuine model improvement.

- **The pass threshold amplifies the effect.** Because an email needs
  4 out of 5 tone criteria to pass, a bias that inflates even one early
  criterion could push borderline emails over the threshold that would
  otherwise fail.

---

## Why This Gap Matters for FDE Work

1. **Reporting benchmark results to a client.** When I present a rubric-judge
   score as evidence of model improvement, I am implicitly claiming the judge
   is unbiased. If position bias is present and I have not audited for it,
   I am overstating confidence in the result. A client engineer who asks
   "how do you know the judge isn't inflating early criteria?" deserves a
   real answer, not a citation of a different bias mitigation.

2. **Designing rubric prompts for new evaluation benchmarks.** Every time
   I build an evaluation for a new client engagement, I will face the same
   design choice: what order do I list the criteria? Without understanding
   the mechanism, I will make that choice arbitrarily and embed an
   unexamined systematic error into every benchmark I ship.

---

## Connection to Existing Work

**Artifact:** `tenacious-bench` — the tone judge prompt inside the
automated scoring evaluator, which presents 5 tone markers in a fixed
sequence and feeds into the reported +25.4% Delta A lift on the held-out
evaluation set.

---

## Explainer Scope

Please cover:

1. The mechanism of position bias in a **single-response rubric judge**
   specifically — not pairwise. How does the autoregressive generation
   process cause criteria listed first to be scored differently than
   criteria listed last?

2. Whether the cross-family defence addresses position bias or only
   self-preference bias — and why the two are distinct failure modes
   that require different mitigations.

3. A detection method I can apply to my existing held-out results to
   check whether position bias is present. What signal in the data would
   confirm it, and what threshold separates material bias from noise?

4. The practical rule: when should I rotate criteria order and re-run,
   and when is disclosure sufficient without re-running?

**Out of scope:** pairwise judge position bias, length bias,
self-preference bias, multi-model judge ensembles.
