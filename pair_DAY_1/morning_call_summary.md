# Day 1 — Morning Call Summary

**Written by:** Meseret Bolled
**Confirmed by:** Gashaw Bekele
**Date:** 2026-05-04

---

## What was ambiguous in the original draft questions

My original question was too broad — I asked generally about "LLM-as-judge
biases" without specifying which type of bias or how it connected to my
specific setup. Gashaw pushed back and asked: are you asking about pairwise
bias or rubric bias, and which dimension of your benchmark is actually
affected? That forced me to name the exact mechanism — criterion ordering in
a single-response rubric judge — and connect it to my specific +25.4% lift
figure and the 15% tone weight.

Gashaw's original question was about whether his Delta A = 0.00 result was
caused by the serving mode. I pushed back and asked: are you asking about
the general case or your specific 0.5B model with r=16, and what does
"silently diverge" mean to you — different top token or just different logit
values? That sharpened his question to focus on the three specific conditions
that cause divergence, not LoRA theory in general.

## How each question was sharpened

**My question changed from:**
A general question about LLM-as-judge biases in my benchmark.

**To:**
A specific question about whether fixed criterion ordering in my
single-response rubric judge inflates pass rates for criteria listed first,
and how to detect that signal in my existing 52 held-out results.

**Gashaw's question changed from:**
A general question about whether merged and unmerged LoRA produce the same
output.

**To:**
A specific question about whether serving mode could be a second-order cause
of his Delta A = 0.00 result, and what conditions cause the two modes to
silently diverge in practice.
