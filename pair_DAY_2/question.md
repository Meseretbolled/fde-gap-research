---
title: Day 2 — My Question
author: Meseret Bolled
date: May 6, 2026
---

# Day 2 — My Question

**Topic:** Agent and tool-use internals
**Asker:** Meseret Bolled
**Explainer:** Hiwot Beyene and Gersum Asfaw
**Date:** May 6, 2026

---

## The Question

In my Week 10 conversion engine, the booking trigger is a Python `any()` check against 10 fixed keywords — "book," "schedule," "call," "meet," "time," "availability," "calendar," "30 min," "happy to," "interested" — that runs before the LLM is called. If the prospect's reply matches, Cal.com slots are fetched and injected into the model's context. If not, the model never learns slots exist.

A prospect who replies "let's see what makes sense timing-wise" or "I'd be open to a conversation" matches zero keywords and receives no booking link — even though intent is clear. The model is never offered a tool schema and never emits a structured function call; HubSpot is called by Python after the LLM responds, not by the LLM itself.

**My question:**

My 10-keyword gate has a systematic miss rate: prospects with high booking intent but non-matching phrasing fall through. If the model were given a `get_booking_link` tool schema and allowed to decide when to call it, would it catch the phrases my keyword list misses — and what exactly changes in what the model sees at the token level that makes that possible?

---

## Why This Question Is Diagnostic

I built the booking trigger as a keyword gate because it was simple and fast. I didn't question it until I read back the code and noticed that "let's see what makes sense timing-wise" — a reply any salesperson would read as a buying signal — matches zero of my 10 keywords. The model never sees a chance to offer booking in that case.

I named the file `hubspot_mcp.py` and described it as using MCP in my README. But the LLM never receives a tool schema and never emits a function call. Python calls HubSpot. This means I do not know whether "function calling" and "context injection" produce different outcomes for ambiguous intent — and I don't know what the model actually receives at the token level that would let it decide versus being told.

---

## Connection to Existing Work

**Artifact:** `conversion-engine/agent/agent_core/conversation_manager.py`

The keyword-matching booking gate on line 43 (`any(kw in reply_text.lower() for kw in ["book","schedule","call","meet","time","availability","calendar","30 min","happy to","interested"])`) fires before the LLM is called. A prospect who replies "let's see what makes sense timing-wise" does not match any keyword and never receives a booking link, even though intent is clear. If function calling were used instead, the model could infer intent from the full reply rather than matching a fixed keyword list.

**Why it matters:** If function calling at the token level produces more reliable booking triggers than Python keyword matching, the conversion rate between "engaged" and "booking offered" stages could be higher than what my current architecture allows. Understanding the real mechanics of tool use would tell me whether this is worth redesigning before a Tenacious pilot.

## Sources I Have Already Read

- My own code: `agent/agent_core/conversation_manager.py`, `agent/agent_core/llm_client.py`, `agent/crm/hubspot_mcp.py`
- OpenRouter API reference (standard OpenAI-compatible `/chat/completions`)
