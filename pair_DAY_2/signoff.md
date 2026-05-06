# Day 2 — Sign-off

**Asker:** Meseret Bolled
**Explainer written by:** Gersum Asfaw (received), Hiwot Beyene (clarifications sent, explainer pending)
**Date:** May 6, 2026

---

## Gap closure verdict

- [x] Closed

---

## What I understand now that I did not before

Before today I thought my booking miss rate was a model problem — the model was "failing to detect intent." Gersum's explainer named the real mechanism: the miss happens before inference. Because Python runs the keyword gate before the LLM is called, the model never receives a tool schema. Continuation B — a structured `get_booking_link` tool call — does not exist as a token path. The model cannot predict something that was never in its context. The fix is not to expand the keyword list; it is to pass the tool schema with `tool_choice="auto"` so the model can choose based on the meaning of the reply. What changes at the token level is that the architecture adds a new valid continuation, and the model's selection is driven by the tool description wording — which means description quality (Layer 2) and scaffold mode (Layer 3) are the two levers I control, not the model's underlying capability. Gersum also correctly identified that the README was misrepresenting the architecture as "agent-initiated booking" when Python was making every decision.
