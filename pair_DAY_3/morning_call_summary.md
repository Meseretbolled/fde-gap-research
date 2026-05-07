# Day 3 — Morning Sync Summary

**Written by:** Meseret Bolled
**Confirmed by:** Kidane Gebremedhin Gidey
**Format:** Slack sync
**Date:** May 7, 2026

---

## What was ambiguous in the original draft questions

Meseret's original draft asked broadly whether "bias in the training signal affects the model's outputs" — which was too wide to answer in 600–1,000 words and did not name a testable prediction. Kidane's original draft asked about SimPO gradient decomposition generally, without anchoring it to the specific failure mode: near-identical pairs whose discriminative tokens are only a handful of tone markers.

## How each question was sharpened

**Meseret's question changed from:**
A general question about whether a biased judge affects DPO training outcomes.

**To:**
A specific question anchored to the +0.1904 Delta A claim: does the position bias in the fixed-order tone compliance judge (0.15 weight) propagate into the fine-tuned model's output distribution — and how would I test whether the lift reflects genuine outreach improvement or a Goodhart artifact? Named artifacts: `scoring_evaluator.py` tone dimension, `methodology_rationale.md` Delta A figure, 159 preference pairs.

**Kidane's question changed from:**
A broad question about where SimPO's preference gradient lands in a sequence.

**To:**
A precise three-part question: (1) how the per-token gradient decomposition works under length normalization, (2) whether near-identical pairs produce gradient concentration or smearing, (3) what cheap diagnostic distinguishes a token-localized critic from a sequence-diffuse one — with the v0.2 body-shape expansion as the named deployment risk.
