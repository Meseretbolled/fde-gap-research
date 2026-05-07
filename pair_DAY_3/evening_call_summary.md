# Day 3 — Evening Sync Summary

**Written by:** Meseret Bolled
**Confirmed by:** Kidane Gebremedhin Gidey
**Format:** Slack sync
**Date:** May 7, 2026

---

## Feedback the asker gave the writer

**Kidane's feedback on Meseret's explainer (answering his question):**
The mask-and-rescore probe landed as immediately runnable — Kidane confirmed the survival ratio threshold (below 20% = token-localized, above 50% = sequence-diffuse) gave him a concrete decision rule he can apply to the shipped adapter before the v0.2 expansion. The asymmetric-in-time framing for the gradient smear (tokens before the discriminative position mostly cancel, tokens after accumulate context-divergence gradient) was called out as the insight he was missing. The per-token attribution heatmap with the sample output made the mechanism visible rather than abstract.

**Meseret's feedback on Kidane's explainer (answering her question):**
The DPO-as-label-faithful-transducer framing closed the gap — the key mechanism is that DPO has no representation of what chosen and rejected *are*, only of the labels, so it absorbs bias as faithfully as signal. The criterion-rotation agreement audit (40 pairs, 5 orderings, majority-vote label vs original label) gave a concrete afternoon-sized experiment with a clear interpretation table (≥0.90 = small bias, 0.70–0.90 = material, <0.70 = retrain). The Goodhart tax framing (proxy eval vs rotation-averaged eval, gap in Delta A units) gave a name to the number she now needs to report.

## What the writer revised after feedback

**Meseret revised her explainer for Kidane:**
- Added the "asymmetric in time" framing to Section 2 to make the pre-vs-post discriminative token distinction explicit
- Added the sample attribution heatmap output to make the token-localized vs smear verdict visually distinguishable
- Added the memo implications section explaining how a high survival ratio changes what the tone-preservation paragraph can claim

**Kidane revised his explainer for Meseret:**
- Added the dual eval experiment (proxy vs gold Delta A) to give a quantified Goodhart tax
- Added the best-of-N attribution diagnostic as a third test
- Clarified that the fix is rotation in the labeling pipeline, not a different model
