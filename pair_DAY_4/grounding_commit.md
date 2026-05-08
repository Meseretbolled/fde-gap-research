# Day 4 — Grounding Commit

**Artifact edited:** `conversion-engine/agent/agent_core/llm_client.py`
**Commit hash:** TBD — fill after committing to conversion-engine repo

---

## What changed and why

Before today, `llm_client.py` made a single `requests.post()` call with no retry logic, no timeout, and no status-aware error handling. A 429 or 503 from OpenRouter would propagate as an unhandled exception up to `conversation_manager.py`, which had no recovery path — the prospect reply was silently dropped.

After reading Abdulaziz's explainer, I added a `_call_with_retry()` wrapper that classifies HTTP responses into three buckets: retryable-with-backoff (429, 408, 500, 502, 503, 504), non-retryable (400, 401, 403), and network errors (retry once). For 429 responses, the wrapper reads the `Retry-After` header if present; otherwise it uses exponential backoff with full jitter capped at 60 seconds, with a maximum of 5 attempts. For 5xx responses, max attempts is 3 with a 10-second cap. The wrapper logs attempt number, status code, and chosen delay on every retry. This does not add a queue or circuit breaker — those are follow-on changes — but it eliminates the silent drop for transient failures, which was the most urgent gap.
