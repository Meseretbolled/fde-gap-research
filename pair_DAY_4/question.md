# Day 4 — My Question

**Topic:** Production patterns — Rate limiting and backpressure for LLM API integrations
**Asker:** Meseret Bolled
**Date:** May 8, 2026

---

## The Question

In my Week 10 conversion engine, `llm_client.py` makes a direct synchronous call to the OpenRouter API for every prospect reply that comes in. There is no retry logic, no rate-limit awareness, and no circuit breaker. The call either succeeds or raises an unhandled exception.

In a real deployment, if 40 prospect replies arrive within the same 5-minute window — which is realistic during a morning outreach burst — all 40 calls hit OpenRouter simultaneously. I do not know what happens at the API level when that occurs: whether OpenRouter queues them, drops them, or returns 429s. I do not know whether a 429 I receive means "wait and retry" or "your request was permanently dropped." I do not know the difference between a rate-limit error I should handle with exponential backoff and a server error I should fail fast on.

**My specific question:**

When `llm_client.py` makes concurrent OpenRouter API calls under bursty load, what is the backpressure mechanism I need to implement — and what is the concrete difference, at the HTTP and retry level, between a 429 rate-limit error that I should retry with exponential backoff versus a 5xx server error I should fail fast on, so that my conversion engine degrades gracefully instead of silently dropping prospect replies?

---

## Why This Question Is Diagnostic

I shipped the conversion engine with a single `requests.post()` call and no error handling beyond a bare `try/except`. I called it "production-ready" in my Week 10 README. But I cannot answer the question: what happens to a prospect reply if the OpenRouter call fails at 9am when 40 replies arrive at once?

This is not a theoretical concern. The conversion engine's entire value is that it responds to every reply. A silent drop — where the API returns a 429, the exception is caught and swallowed, and the prospect never hears back — would be invisible in my current logging and would cost a real booking. I have no instrumentation that would even tell me this happened.

The gap is in `llm_client.py`. I built the happy path. I did not build the failure path.

---

## Connection to Existing Work

**Artifact:** `conversion-engine/agent/agent_core/llm_client.py`

My `llm_client.py` makes a single `requests.post()` call to OpenRouter's `/chat/completions` endpoint with no retry logic, no timeout, and no rate-limit handling. If the call fails with a 429 or 503, the exception propagates up to `conversation_manager.py` which has no recovery logic either — the prospect reply is silently dropped and no booking is offered.

**Why it matters:** If I understand how backpressure and retry logic work at the HTTP level for LLM APIs, I can add a retry wrapper with exponential backoff and jitter to `llm_client.py` that handles transient failures without dropping replies. This would change the conversion engine from a system that fails silently under load to one that degrades gracefully — which is the difference between a demo and a production deployment. Any FDE building a client-facing LLM integration faces this exact design choice.

## Sources I Have Already Read

- My own code: `agent/agent_core/llm_client.py`, `agent/agent_core/conversation_manager.py`
- OpenRouter API reference — lists rate limit headers (`X-RateLimit-Limit`, `X-RateLimit-Remaining`, `Retry-After`) but does not explain the retry strategy
