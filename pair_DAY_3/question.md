# Day 3 — My Question

**Topic:** Training and post-training mechanics
**Asker:** Meseret Bolled
**Explainer:** Kidane Gebremedhin
**Date:** May 7, 2026

---

## The Question

In my Week 11 tenacious-bench, I constructed 159 DPO preference pairs from hand-reviewed agent traces. The "chosen" outputs are traces that passed my scoring evaluator; the "rejected" outputs are traces that failed. The scoring evaluator's tone compliance dimension — weighted at 0.15 — uses an LLM judge that scores 5 tone markers in a fixed order: direct, grounded, honest, professional, non_condescending.

Day 1 research showed this fixed ordering creates position bias: the judge inflates scores for criteria listed first because autoregressive token generation makes earlier criteria easier to attend to consistently. If the preference labels used to train the DPO model were partially determined by that biased judge, the fine-tuned model may have learned to satisfy first-listed tone criteria more reliably than last-listed ones — not because those criteria matter more, but because the training signal said they did.

**My question:**

When a biased LLM judge is used to label preference pairs for DPO training, does the bias in the labels propagate into the fine-tuned model's output distribution — and how would I test whether my reported Delta A lift of +0.1904 reflects genuine outreach improvement or optimization against a systematically skewed signal?

---

## Why This Question Is Diagnostic

I reported a Delta A of +0.1904 (95% CI [0.1115, 0.2788], p=0.0000) as the core evidence that DPO training improved my agent. That claim rests on the assumption that the preference pairs captured genuine quality differences. But the tone compliance scoring — which influenced 15% of every preference label — came from a judge with a known ordering effect I had not measured at training time.

I cannot currently defend whether the lift reflects better outreach or whether the model learned to produce outputs that score well on a biased rubric. Those are different things: the first is a capability improvement, the second is Goodhart's Law in post-training — the model optimised against a measure correlated with quality but not identical to it.

---

## Connection to Existing Work

**Artifact:** `tenacious-bench/src/evaluation/scoring_evaluator.py` — tone compliance dimension (weight 0.15) uses a fixed-order LLM judge across all 159 preference pair labels, and `tenacious-bench/methodology_rationale.md` where the Delta A = +0.1904 claim is the headline evidence for DPO effectiveness.

**Why it matters:** If bias propagated into the training signal, the fix is to run a criterion rotation audit on the preference pairs, quantify the skew, and either add a caveat to the methodology or re-label the affected pairs before the next training run. Understanding the propagation mechanism tells me which of those actions is actually necessary.

---

## Sources I Have Already Read

- `tenacious-bench/src/evaluation/scoring_evaluator.py`
- `tenacious-bench/methodology_rationale.md`
- Wang et al. (2023) — position bias in LLM-as-judge rubrics (from Day 1 research)
