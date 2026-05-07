# Portfolio Update — Week 12

**Author:** Meseret Bolled
**For:** FDE Hiring Manager

---

## Overview

Five grounding commits made to my Week 10/11 work as a direct result of gaps closed in Week 12.

---

## The Five Commits

### Commit 1 — Day 1
- **Artifact:** `tenacious-bench/methodology_rationale.md`
- **What changed:** Added a named limitation paragraph distinguishing self-preference bias (mitigated by cross-family evaluation) from position bias (unaudited). Explains the mechanism — primacy effects and consistency pressure in autoregressive token generation — and recommends a rotation audit before reporting tone scores as bias-free. Cites Wang et al. (2023) and Zheng et al. (2023).
- **Why it matters:** My benchmark's +25.4% lift claim previously implied the judge was fully unbiased. It now discloses a specific, named limitation with an estimated impact and a concrete path to fixing it. A client engineer or hiring manager reviewing the methodology can see the gap was identified and handled honestly, not papered over.

### Commit 2 — Day 2
- **Artifact:** `conversion-engine/agent/agent_core/conversation_manager.py` and `README.md`
- **What changed:** Added an architectural comment above the 10-keyword booking gate labelling it as a Layer 3 scaffold decision (not model tool choice), naming the specific phrases it misses ("let's see what makes sense timing-wise," "I'd be open to a conversation"), and documenting the upgrade path to a `get_booking_link` tool schema with `tool_choice="auto"`. Updated README and method.md to replace "agent-initiated booking" with "Python-gated scaffold routing."
- **Why it matters:** My Week 10 README described the booking flow as agent-initiated when Python was making every decision before the model was called. A hiring manager or client engineer reading the architecture now sees an honest description of what the system does and a concrete named path to model-driven improvement. The comment also means any future engineer who touches that file understands why the keyword list is not the right fix.

### Commit 3 — Day 3
- **Artifact:** `tenacious-bench/methodology_rationale.md`
- **What changed:** Added a named limitation paragraph directly below the +0.1904 Delta A figure (95% CI [0.1115, 0.2788], p=0.0000). The paragraph states: the tone compliance dimension (weight 0.15) uses a fixed-order LLM judge known to produce primacy effects; the criterion-rotation agreement audit has not yet been run; until the audit is complete, Delta A should be read as an upper bound on genuine quality improvement with an unknown Goodhart tax. It specifies the exact audit needed (40-pair sample, 5 orderings, majority-vote label vs original label) and the interpretation thresholds: ≥0.90 = defensible with caveat, 0.70–0.90 = material bias warranting disclosure, <0.70 = retrain with rotation before publishing the number. Cites Rafailov et al. (2023) for the DPO label-propagation mechanism and Gao et al. (2023) for the proxy/gold reward divergence.
- **Why it matters:** My Week 11 methodology previously reported Delta A as the headline evidence of DPO training improvement without disclosing that the label source for one training dimension had a known and unaudited bias. A hiring manager or client engineer reading the document now sees: the number, the uncertainty around it, the specific mechanism that could inflate it, and a concrete path to resolving the inflation. The Goodhart tax is named, not papered over. This is the difference between a benchmark that implies completeness and one that discloses its limits.

### Commit 4 — Day 4
- **Artifact:**
- **What changed:**
- **Why it matters:**

### Commit 5 — Day 5
- **Artifact:**
- **What changed:**
- **Why it matters:**

---

## Collective Impact

<!-- One paragraph for a hiring manager: what is defensibly better about your Week 10/11 portfolio now vs before Week 12 -->
