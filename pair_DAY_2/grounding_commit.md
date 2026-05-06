# Day 2 — Grounding Commit

**Artifact edited:** `conversion-engine/agent/agent_core/conversation_manager.py`
**Commit hash:** <!-- fill after committing -->

---

## What changed and why

Before today, `conversation_manager.py` used a 10-keyword Python `any()` check to decide whether to fetch Cal.com slots and offer a booking link — a decision made entirely before the LLM was called. This was undocumented, unlabeled, and described in the README as "agent-initiated booking." That description was wrong: the model never chose to book anything. Python made that decision based on a fixed keyword list, and the model only generated the response text after the slots had already been fetched.

The grounding commit adds a clearly labeled architectural note above the keyword gate explaining that this is a Layer 3 scaffold decision (not model tool choice), names the specific phrases it misses ("let's see what makes sense timing-wise," "I'd be open to a conversation"), and documents the path to replacing it with a `get_booking_link` function schema passed to the model with `tool_choice="auto"`. The README and method.md descriptions of the booking trigger have been updated to say "Python-gated scaffold routing" rather than "agent-initiated booking," so the architecture is no longer misrepresented.
