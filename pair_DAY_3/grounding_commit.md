# Day 3 — Grounding Commit

**Artifact edited:** `tenacious-bench/methodology_rationale.md`
**Commit hash:** <!-- fill after committing -->

---

## What changed and why

Before today, `methodology_rationale.md` reported Delta A = +0.1904 (95% CI [0.1115, 0.2788], p=0.0000) as the headline evidence that DPO training improved the agent, with a brief note that the tone compliance dimension uses a fixed-order LLM judge. The document did not acknowledge that the fixed ordering creates position bias, did not quantify what fraction of the 159 preference pair labels may have been nudged by ordering effects, and did not distinguish between genuine outreach improvement and Goodhart optimization against a biased proxy.

The grounding commit adds a named limitation paragraph directly below the Delta A figure stating: the tone compliance score (weight 0.15) was computed with a fixed-order rubric known to produce primacy effects; the criterion-rotation agreement audit has not yet been run; until the audit is complete, the Delta A figure should be read as an upper bound on genuine quality improvement, with an unknown Goodhart tax. The paragraph also specifies the exact audit needed (40-pair sample, 5 orderings, majority-vote comparison) and the interpretation thresholds (≥0.90 = defensible with caveat, 0.70–0.90 = material bias, <0.70 = retrain with rotation). This converts a known-but-unnamed limitation into an explicit, bounded disclosure with a concrete remediation path.
