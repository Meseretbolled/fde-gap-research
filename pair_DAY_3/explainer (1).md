# When the judge has a bias, DPO learns it — how a position-biased rubric becomes a position-biased policy

*Explainer for Meseret's Day-3 question on whether the +0.1904 Delta A on her Week 11 tenacious-bench reflects genuine outreach improvement or optimization against a position-biased tone-compliance judge.*

You wrote: *"When a biased LLM judge is used to label preference pairs for DPO training, does the bias in the labels propagate into the fine-tuned model's output distribution — and how would I test whether my reported Delta A lift of +0.1904 reflects genuine outreach improvement or optimization against a systematically skewed signal?"*

Short answer: **yes, it propagates — DPO is a faithful transducer of whatever distinction the (chosen, rejected) labels encode**, and a fixed-order tone judge encodes a distinction that is partly "satisfied the first-listed markers" rather than "wrote a better email." The lift is not necessarily fake; it almost certainly contains *some* genuine signal because tone compliance is only weighted 0.15. But you cannot read the headline number alone — you need a small criterion-rotation audit and a dual eval to decompose the lift into "real quality" vs "Goodhart tax." Both are doable in a few hours.

## The mechanism — why DPO can't tell label bias from label signal

DPO's loss, from [Rafailov et al., 2023](https://arxiv.org/abs/2305.18290), is

```
L_DPO = -E_(x, y_w, y_l) [ log σ ( β · log[π_θ(y_w|x)/π_ref(y_w|x)]
                                  - β · log[π_θ(y_l|x)/π_ref(y_l|x)] ) ]
```

The gradient pushes up `log π_θ(y_w|x)` and pushes down `log π_θ(y_l|x)`, scaled by how confident the implicit reward already is in the wrong direction (§4 of the DPO paper, the "implicit reward" derivation). The objective contains no representation of what `y_w` and `y_l` *are*; it contains only the labels. DPO is a structurally label-faithful learner — it absorbs whatever structure the preference distribution carries, including artifacts.

Now look at what your tone judge actually measures. With five markers scored in fixed order — direct, grounded, honest, professional, non_condescending — the autoregressive judge attends most reliably to the first criterion's evidence, conditions every later score on the earlier ones, and sharpens the distribution toward early criteria as a function of position alone ([Wang et al., 2023, "Large Language Models are not Fair Evaluators"](https://arxiv.org/abs/2305.17926); [Zheng et al., 2023, MT-Bench](https://arxiv.org/abs/2306.05685), §5 on judge biases). So a draft that nails *direct* and *grounded* but is mildly condescending will tend to score higher than a draft that is courteous but slightly indirect — even if the second is better outreach.

When that biased score crosses the pass/fail threshold and lands in your `chosen`/`rejected` split, every preference pair where ordering flipped the label injects a small, consistent direction into your training set. Across 159 pairs, even a modest contamination rate (say 15–25% of labels nudged by ordering) produces a coherent gradient. DPO will learn *that* gradient as faithfully as it learns the "real" outreach-quality gradient, because at the loss level they are indistinguishable.

## What the +0.1904 is actually measuring

There are two regimes:

- **Eval shares the bias.** If your Delta A eval also runs the fixed-order rubric, it scores the trained model's outputs through the same skew that produced the labels. The model is rewarded twice for the same artifact: once during training, once at evaluation. Some part of +0.1904 is then tautological — it is the gradient you already added showing up where you already pointed the camera. This is the canonical proxy-vs-gold-reward gap from [Gao et al., 2023, "Scaling Laws for Reward Model Overoptimization"](https://arxiv.org/abs/2210.10760), Figure 1: proxy reward keeps rising while gold reward plateaus and eventually drops.
- **Eval is independent.** If the eval rubric is different — different judge, different ordering, or human-scored — then the lift is what the model actually transferred to a new measurement, and it is largely defensible. Tone is only 0.15 of your weight, so most of Delta A is signal-grounding and bench-honesty either way; the question is how much of the *remaining* fraction the bias eats.

You cannot tell which regime you're in from the number alone. You need to *measure* the gap. That is what the audit below does.

## The audit that distinguishes real lift from Goodhart

Two cheap experiments, both small enough to run in an afternoon on the held-out partition.

**Experiment 1 — criterion-rotation agreement on the labels.** Take a sample of, say, 40 of your 159 preference pairs. Re-run the tone-compliance judge five times per pair, each time with a different marker order (a Latin square over the 5 positions). Take majority vote across orderings as the "rotation-fair" label. Compute agreement with the original fixed-order label.

```
agreement_rate = (rotation_fair_label == original_label).mean()
```

Interpretation:
- ≥0.90 — bias is real but small; your +0.1904 is mostly real lift; add a one-paragraph caveat to `methodology_rationale.md` and move on.
- 0.70–0.90 — bias is material; some fraction of pairs were mis-labeled by ordering. Quantify the directional skew (which markers won when?), re-label the audited 40, and either retrain with the corrected pairs or report a corrected Delta A.
- <0.70 — the labels are substantially an artifact of ordering. Retrain with rotation in the labeling pipeline before drawing further conclusions.

**Experiment 2 — dual eval (proxy vs gold).** Score the trained model's held-out outputs twice: once with the original fixed-order rubric (proxy), once with rotation-averaged scoring (closer to gold). Report `Δ_fixed − Δ_rotation`. That gap, in Delta A units, is the **Goodhart tax** — the part of your lift that lives only in the biased measurement frame. This mirrors Gao et al.'s proxy/gold decomposition for reward-model overoptimization, applied to your judge instead of an explicit RM.

If you want one more diagnostic: run a **best-of-N attribution** ([Skalse et al., 2022, "Defining and Characterizing Reward Hacking"](https://arxiv.org/abs/2209.13085) frames why this works). Sample N=8 candidates per held-out prospect from the trained model. Score all 8 with the proxy and the gold. If proxy/gold disagreement is *higher* on the model's top-1 picks than on a random pick, the model has learned to exploit the bias — that is hacking, not generalization.

## Connect the dots

**Reward hacking, Goodhart's Law, and proxy/gold gap are the same phenomenon under three vocabularies.** Skalse et al. give the formal definition (a proxy reward is "hackable" when there exist policies improving on the proxy without improving on the true objective). Gao et al. give the empirical scaling law showing the gap *grows* with optimization pressure. Your DPO run is exactly such an optimization pressure on a proxy with a known bias direction. The audit above is the standard way of measuring the gap in the absence of an explicit gold judge.

**Position bias is one of three known LLM-judge artifacts**; the other two (length bias and self-preference bias, also catalogued in Wang et al. 2023 and Zheng et al. 2023) likely also touch your pairs. If you're already auditing position, run a quick length-bias check too — score `len(y_w) − len(y_l)` correlation with the label and see if it's significant. Free, and often surprising.

**The fix is rotation in the labeling pipeline, not a different model.** A criterion-rotation wrapper around your judge call (`random.shuffle(criteria)` per call, average scores across calls if you want stability) removes the bias at source. It costs you a 5× judge-call multiplier on the labels, which on 159 pairs is small.

## Pointers

- **Direct Preference Optimization** (Rafailov et al., 2023) — https://arxiv.org/abs/2305.18290 (the gradient derivation, §4)
- **Scaling Laws for Reward Model Overoptimization** (Gao et al., 2023) — https://arxiv.org/abs/2210.10760 (the proxy/gold gap, Fig. 1)
- **LLMs are not Fair Evaluators** (Wang et al., 2023) — https://arxiv.org/abs/2305.17926 (position bias mechanism)
- **MT-Bench / Judging LLM-as-a-Judge** (Zheng et al., 2023) — https://arxiv.org/abs/2306.05685 (catalogue of judge biases + mitigation patterns)
- **Defining and Characterizing Reward Hacking** (Skalse et al., 2022) — https://arxiv.org/abs/2209.13085 (the formal frame for what your audit is testing)
