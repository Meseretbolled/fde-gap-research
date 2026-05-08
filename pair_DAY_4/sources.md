# Day 4 — Sources

## Canonical Papers / Primary Sources (for Meseret's explainer on bootstrap CIs)

1. **An Introduction to the Bootstrap**
   - Authors: Bradley Efron, Robert J. Tibshirani
   - Link: https://doi.org/10.1007/978-1-4899-4541-9
   - Why I used it: Foundational text defining bootstrap CI construction (Chapter 6) and the two-sample comparison underpinning paired bootstrap (Chapter 11). Primary source for the resampling mechanics described in the explainer.

2. **An Empirical Investigation of Statistical Significance in NLP**
   - Authors: Taylor Berg-Kirkpatrick, David Burkett, Dan Klein
   - Link: https://aclanthology.org/D12-1091/
   - Why I used it: Canonical paper demonstrating that unpaired significance tests overstate certainty in NLP evaluation and that paired bootstrap corrects for task-difficulty confounding. Directly applicable to agent benchmark comparisons using `eval/score_log.json`.

---

## Canonical Sources (for Abdulaziz's explainer on backpressure — received by Meseret)

3. **OpenRouter API Reference — Errors and Debugging**
   - Link: https://openrouter.ai/docs/api/reference/errors-and-debugging
   - Why it was used: Authoritative documentation defining OpenRouter's 429, 502, and 503 semantics and what they mean for client retry behavior.

4. **MDN Web Docs — HTTP Status 429 / 503 / Retry-After**
   - Links: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/429 · https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/503 · https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Retry-After
   - Why it was used: Authoritative HTTP semantics definitions distinguishing client-pressure signals (429) from upstream-health signals (5xx) and specifying Retry-After header behavior.

---

## Tool or Pattern Used

- **Tool:** `numpy.random.default_rng` with manual resampling loop (bootstrap CI implementation)
- **What I did with it:** Implemented both simple and paired bootstrap over synthetic score_log data matching Abdulaziz's format, verified that paired bootstrap CI excludes zero while unpaired CIs overlap for the same underlying data.
- **What I observed:** With N=30 tasks and a genuine +13 pp improvement, the unpaired bootstrap CIs overlapped (not defensible), while the paired bootstrap CI excluded zero (defensible) — confirming the paired method is required for same-task comparisons.

---

## Follow-on Reading

- Dror et al. (2018) "The Hitchhiker's Guide to Testing Statistical Significance in Natural Language Processing" — extends Berg-Kirkpatrick with practical guidance on sample size requirements for NLP benchmarks.
