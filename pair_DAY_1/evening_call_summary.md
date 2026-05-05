# Day 1 — Evening Call Summary

**Written by:** Meseret Bolled
**Confirmed by:** Gashaw Bekele
**Date:** 2026-05-04
**Call duration:** ~30 minutes

---

## Feedback the asker gave the writer

**On Gashaw's explainer (position bias in rubric judges — written for Meseret):**
The mechanism section landed well — the autoregressive generation explanation and the distinction between pairwise and single-response position bias was clear. Two things did not land initially: (1) the audit procedure was described in prose but no concrete code snippet was given, making it hard to see exactly what to run on the 52 held-out results; (2) the threshold for "material bias" vs noise was named but not quantified — Meseret asked "what number would tell me this is real?" and the original draft did not answer that directly.

**On Meseret's explainer (LoRA merged vs unmerged — written for Gashaw):**
Gashaw confirmed the three divergence conditions (fp16 rounding, α/r scaling bug, dropout at inference) were clear and in the right order of likelihood. He flagged that the final section on his specific Delta A = 0.00 result was helpful but should be more explicit that serving mode is ruled out as the primary cause — the original wording left some ambiguity about whether to check serving mode first or the backbone capacity bottleneck first.

## What the writer revised

**Gashaw revised his explainer** to add a concrete pandas-based audit snippet showing how to compute per-criterion pass rates across the 52 tasks and flag criteria whose pass rate exceeds the mean by more than one standard deviation as a positive signal for position bias.

**Meseret revised her explainer** to add a numbered debug order in the practical FDE rule section, making explicit that `model.eval()` and the scaling factor check should run before concluding the backbone is the bottleneck — clarifying the Gashaw-specific conclusion in Section 4.
