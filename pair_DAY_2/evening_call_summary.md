# Day 2 — Evening Sync Summary

**Written by:** Meseret Bolled
**Confirmed by:** Gersum Asfaw, Hiwot Beyene
**Format:** Slack (no Google Meet today)
**Date:** May 6, 2026

---

## Feedback the asker gave the writer

**Gersum's feedback on Meseret's explainer:**
The three-layer framework landed clearly — Gersum confirmed the attribution decision tree is directly usable for debugging his pipeline. The section on hidden reasoning tokens was flagged as particularly useful because he had been planning to log `<thinking>` blocks and did not know they were unavailable on most OpenRouter models. The MCP vs function calling clarification resolved a terminology confusion he had carried since Week 10.

**Hiwot's feedback on Meseret's explainer:**
The in-token tool representation section answered the specific question she had about what the model "sees" before selecting a tool — she had not known the description field is serialized as plain text into the model's input. The `detect_drift()` function and the Turn 4 warning for her pipeline were called out as immediately actionable. The safe vs unsafe logging table gave her a clear rule for what to store without needing to re-read the policy document.

## What the writer revised after feedback

- Added the MCP vs function calling clarification note to both Part 1 and Part 2 after both Gersum and Hiwot flagged that their questions used "MCP" but their implementations use OpenRouter's OpenAI-compatible API.
- Made the attribution decision tree in Part 1 a standalone ASCII diagram so it can be read without the surrounding prose.
- Added the turn-by-turn table in Part 2 (Hiwot's section) mapping each stage of her pipeline to the recommended `tool_choice` mode and what to watch at that turn.
