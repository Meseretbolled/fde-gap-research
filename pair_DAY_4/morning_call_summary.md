# Day 4 — Morning Call Summary

**Written by:** Meseret Bolled
**Confirmed by:** Abdulaziz
**Date:** May 8, 2026
**Call duration:** No morning call — questions finalised asynchronously via Slack before midday.

---

## What was ambiguous in the original draft questions

My original question asked broadly about "what happens when my API call fails under load" without naming the specific mechanism I was missing. Abdulaziz pushed back and asked: are you asking about the retry policy, the concurrency model, or the queue design — because those are three different things and one explainer cannot close all three. That forced me to narrow to the concrete HTTP-level difference between 429 and 5xx, which is the decision point that determines whether to retry or degrade.

Abdulaziz's original question was broad — "how do I report statistical uncertainty for my benchmark?" I pushed back: are you comparing two versions or characterising one version, because those require different methods? He sharpened it to ask specifically when paired bootstrap is required over simple bootstrap, grounded in his `score_log.json` and the same-task requirement.

## How each question was sharpened

**My question changed from:**
A general question about what to do when OpenRouter API calls fail under load.

**To:**
A specific question about the concrete HTTP-level difference between a 429 rate-limit error and a 5xx server error, and what backpressure mechanism to implement in `llm_client.py` so prospect replies are never silently dropped.

**Abdulaziz's question changed from:**
A general question about reporting statistical uncertainty for benchmark results.

**To:**
A specific question about when simple bootstrap is sufficient versus when paired bootstrap is required for comparing two versions of the conversion agent — grounded in whether the same task IDs appear in both versions of `eval/score_log.json`.
