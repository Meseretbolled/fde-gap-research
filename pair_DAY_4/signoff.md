# Day 4 — Sign-off

**Asker:** Meseret Bolled
**Explainer written by:** Abdulaziz
**Date:** May 8, 2026

---

## Gap closure verdict

- [x] Closed
- [ ] Partially closed
- [ ] Not closed

---

## What I understand now that I did not before

Before today I thought the failure handling problem in `llm_client.py` was a retry problem — add exponential backoff and it is solved. The explainer changed my mental model in two ways. First, I now understand that retry logic alone is insufficient if concurrency is unbounded: 40 simultaneous retries recreate the same overload that caused the 429 in the first place. The real fix is a bounded concurrency limit in front of the OpenRouter call, so inbound burst pressure never directly becomes outbound API pressure. Second, I now understand the precise difference between 429 and 5xx: a 429 is a client-pressure signal that says "slow down and retry more patiently," while a 5xx is an upstream-health signal that should trigger a circuit breaker after a small retry budget. Treating them identically — either both "retry forever" or both "fail fast" — is wrong in both directions. The concrete change this produces in `llm_client.py` is a status-aware retry classifier, not a single catch-all `except` block.
