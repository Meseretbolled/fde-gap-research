# Day 3 — Sign-off

**Asker:** Meseret Bolled
**Explainer written by:** Kidane Gebremedhin Gidey
**Date:** May 7, 2026

---

## Gap closure verdict

- [x] Closed

---

## What I understand now that I did not before

Before today I reported a Delta A of +0.1904 as evidence that DPO training improved my agent and treated the number as defensible. What I did not understand is why DPO cannot distinguish label bias from label signal — the loss has no representation of what chosen and rejected *are*, only of their labels. So the position bias in my fixed-order tone compliance judge (which inflates scores for criteria listed first) produces a consistent gradient direction across the 159 pairs, and DPO faithfully learns that gradient alongside the genuine outreach-quality signal.

Kidane's explainer named the specific test I need: a criterion-rotation agreement audit on a sample of 40 pairs, re-running the judge five times per pair with different marker orderings and comparing majority-vote labels to the original fixed-order labels. An agreement rate below 0.90 means the bias is material and the Goodhart tax needs to be reported alongside the Delta A figure. An agreement rate below 0.70 means I need to retrain with rotation in the labeling pipeline before the number is defensible. I now know exactly what to run and what the result means — which is what gap closure requires.
