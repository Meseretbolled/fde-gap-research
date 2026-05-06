# Clarification Replies — For Hiwot Beyene

**From:** Meseret Bolled
**Date:** May 6, 2026

---

**1. Booking path:**
Option A — one LLM call. The keyword gate runs first, fetches Cal.com slots if it fires, then injects them into context before a single `chat()` call. There is no multi-turn loop.

---

**2. What comparison I want:**
Option 1 as primary — same model, two scaffold policies: (a) keyword gate injects slot text into the prompt vs (b) model emits `get_booking_link` and slot text comes back as a tool result message. Option 3 (recall on ambiguous replies) as secondary — that is the real business motivation.

---

**3. Precision vs recall worry:**
Option b — missing soft-intent replies is the bigger concern. False positives (offering booking too early) are less damaging than missing an engaged prospect who just phrased it differently.

---

**4. HubSpot order of operations:**

1. Inbound reply arrives
2. Python keyword `any()` runs
3. LLM called (with or without slots in context)
4. Python calls HubSpot after the LLM response using the SDK directly

The LLM never emits a function call today. I want HubSpot writes to stay deterministic post-LLM — I do not need the model to drive that.

---

**5. Token level — what I want spelled out:**
Option a + c — a concrete diff of the message roles (system / user / assistant / tool) between keyword-injection and tool-calling flows, lightweight, no heavy math needed.

---

**6. Example phrases that miss the keyword gate:**

- "let's see what makes sense timing-wise"
- "I'd be open to a conversation"
- "what does your process look like"

Borderline that fires too eagerly: "I don't have time for this" — contains "time" which matches the keyword list but signals the opposite of booking intent.
