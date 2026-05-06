# Day 2 — Sign-off

**Asker:** Meseret Bolled
**Explainer written by:** Gersum Asfaw + Hiwot Beyene
**Date:** May 6, 2026

---

## Gap closure verdict

- [x] Closed

---

## What I understand now that I did not before

Before today I thought my booking miss rate was a model problem — the model was "failing to detect intent." Both explainers independently named the same root cause: the miss happens before inference. Python runs the keyword gate before the LLM is called, so the model never receives a tool schema and Continuation B (a structured `get_booking_link` tool call) does not exist as a token path. The model cannot predict something that was never in its context.

Gersum named the architectural mechanism: the keyword gate is a Layer 3 scaffold decision, not model tool choice, and the fix is to pass the tool schema with `tool_choice="auto"` so the model can choose based on meaning rather than lexical match.

Hiwot added the concrete transcript-level picture I was missing: the difference between the two paths is visible in the message roles themselves. In the keyword-gate path, calendar facts appear as plain text pasted into the user message by Python — or not at all if no keyword matched. In the tool-calling path, they appear as a `tool` role message after the model emitted a `tool_calls` payload, which means the model requested the data based on the semantic meaning of the reply. Hiwot also named the engineering cost I had not thought about: native function calling requires two LLM forward passes (call → tool result → final answer), not one, which is a real latency and cost consideration before a pilot.

Gap is fully closed. The next step is a prototype two-step branch on a dev model and a recall measurement against a frozen list of soft-intent phrases.
