# Day 1 — Grounding Commit

**Artifact edited:** `tenacious-bench/methodology_rationale.md`
**Commit hash:** TBD — fill after committing to tenacious-bench

---

## What changed and why

Before today, my methodology defended the tone judge on one ground only:
using a different model family from the agent prevents self-preference bias.
That defence was cited as complete.

After Gashaw's explainer on position bias, I understood that self-preference
bias and position bias are two separate failure modes requiring two separate
mitigations. My cross-family defence addresses only the first. The fixed
ordering of my five tone criteria — direct → grounded → honest → professional
→ non_condescending — has never been audited for inter-criteria position bias,
where criteria listed first receive inflated scores due to primacy effects and
consistency pressure in autoregressive token generation.

I added a paragraph to `methodology_rationale.md` explicitly naming position
bias as a known limitation, explaining why the cross-family defence does not
cover it, and recommending a rotation audit before reporting tone scores as
bias-free. The estimated impact on the overall weighted score is below 0.03
points, but the effect on individual criterion pass rates is unquantified and
should be disclosed to any client reviewing the benchmark results.
