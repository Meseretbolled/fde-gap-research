# Day 1 — Sign-off

**Asker:** Meseret Bolled
**Explainer written by:** Gashaw Bekele
**Date:** 2026-05-04

---

## Gap closure verdict

- [ ] Closed
- [x] Partially closed
- [ ] Not closed

---

## What I understand now that I did not before

Before today I defended my tone judge with a single argument: using Qwen3 to judge a Qwen2.5-based agent prevents self-preference bias. I believed that was a complete defence. Gashaw's explainer made clear that self-preference bias and position bias are two entirely separate failure modes — the cross-family defence addresses only the first and says nothing about the second. I now understand that in a single-response rubric judge, criteria listed first receive inflated pass rates not because the emails are genuinely better on those criteria, but because autoregressive generation creates primacy pressure: the model's early binary decisions (direct, grounded) anchor the consistency of later ones (non_condescending). My five tone criteria have always been in fixed order and have never been audited for this effect. The gap is partially closed because I understand the mechanism and have named it as a limitation in methodology_rationale.md, but I have not yet run the rotation audit on my 52 held-out results to quantify whether the bias is material.
