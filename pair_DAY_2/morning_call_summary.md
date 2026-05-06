# Day 2 — Morning Sync Summary

**Written by:** Meseret Bolled
**Confirmed by:** Gersum Asfaw, Hiwot Beyene
**Format:** Slack (no Google Meet today)
**Date:** May 6, 2026

---

## What was ambiguous in the original draft questions

The original draft of Meseret's question asked broadly whether function calling and keyword matching "produce different outcomes" — which was too vague to answer in 600–1,000 words. It did not name a specific failure mode or a measurable gap. Gersum's and Hiwot's questions both used the term "MCP tool calls" without clarifying whether their implementations actually used MCP or the OpenAI-compatible `tools` parameter — an important distinction that needed surfacing before writing the explainer.

## How each question was sharpened

**Meseret's question changed from:**
A general comparison of keyword matching vs function calling.

**To:**
A specific failure mode — the 10-keyword gate in `conversation_manager.py` has a systematic miss rate for high-intent prospects whose phrasing does not match any keyword. The question now asks what changes at the token level when the model is given a `get_booking_link` tool schema instead, and whether it catches the phrases the keyword list misses.

**Gersum's question changed from:**
A broad question about MCP and multi-turn orchestration.

**To:**
A precise three-layer attribution question: model tool choice (Layer 1), description quality effect (Layer 2), scaffold policy effect (Layer 3) — with the specific requirement that hidden reasoning tokens are not part of the answer.

**Hiwot's question changed from:**
Similar framing to Gersum's but without a specific pipeline anchoring it.

**To:**
The same three-layer question grounded explicitly in her `enrich → compose → send → reply → book` pipeline, with additional focus on multi-turn planning drift and the distinction between hidden scratch work and auditable output.
