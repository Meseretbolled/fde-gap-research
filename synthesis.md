# Week 12 Synthesis

**Author:** Meseret Bolled
**Submitted:** <!-- date -->

---

## The 10 Gaps Closed

### Gaps I Named (questions I asked, my partner explained)

| Day | My Question | Gap Closed? |
|-----|------------|-------------|
| 1 | Does fixed criterion ordering in my single-response rubric judge inflate scores for criteria listed first via position bias — and does that distortion touch my +25.4% lift? | Partially closed — mechanism understood, rotation audit not yet run |
| 2 | My 10-keyword booking gate has a systematic miss rate — prospects with high intent but non-matching phrasing fall through. Would giving the model a `get_booking_link` tool schema catch the phrases my list misses, and what changes at the token level? | Closed — Gersum's explainer named the mechanism: the architecture changes what the model is allowed to predict |
| 3 | TBD | TBD |
| 4 | TBD | TBD |
| 5 | TBD | TBD |

### Gaps I Researched (my partner's questions, I explained)

| Day | Partner's Question | Their Sign-off |
|-----|-------------------|---------------|
| 1 | Are merged and unmerged LoRA weights mathematically guaranteed to produce identical logits, or are there conditions where they silently diverge? (Gashaw Bekele) | TBD — pending Gashaw's signoff |
| 2 | How to distinguish model tool choice, description quality effects, and scaffold policy in multi-turn traces without relying on hidden reasoning tokens (Gersum Asfaw + Hiwot Beyene) | Closed — both confirmed mechanism and trace schema landed |
| 3 | TBD | TBD |
| 4 | TBD | TBD |
| 5 | TBD | TBD |

---

## The Most Surprising Thing I Learned

**Day 1:** I had one defence for my tone judge — cross-family evaluation prevents self-preference bias — and I believed it was complete. The most surprising thing I learned is that self-preference bias and position bias are two entirely separate failure modes that require two entirely different mitigations. My cross-family defence says nothing about whether criteria listed first in the prompt receive inflated scores due to how autoregressive models generate tokens sequentially. I had been treating one mitigation as two, and I did not know they were different until today.

**Day 2:** I thought my keyword gate was a model problem — the model was "failing to detect intent." Gersum's explainer showed the miss happens before inference. The model never receives a tool schema so Continuation B (a tool call) does not exist as a token path. The gap is architectural, not a model capability issue. The surprising thing is how confidently I had described the booking flow as "agent-initiated" when Python was making every decision.

---

## Canonical Reading List

See [canonical_list.md](canonical_list.md) for the full annotated list.

**Day 1 additions:**
- Hu et al. (2021) — LoRA original paper
- Wang et al. (2023) — LLM-as-judge position bias
- Zheng et al. (2023) — MT-Bench and Chatbot Arena judge calibration

**Day 2 additions:**
- OpenAI Function Calling docs — canonical reference for `finish_reason`, `tool_choice` modes, and tool schema format
- Yao et al. (2023) ReAct — reasoning-action loop and the basis for separating model choice from scaffold execution
- Schick et al. (2023) Toolformer — how tool-use is learned, explains why description quality affects selection probability

---

## How This Week Changed My Work

**Day 1:** Added a named limitation paragraph to `tenacious-bench/methodology_rationale.md` distinguishing self-preference bias (mitigated) from position bias (unaudited). The +25.4% lift claim is now accompanied by an honest disclosure rather than an implied completeness.

**Day 2:** Added an architectural comment to `conversion-engine/agent/agent_core/conversation_manager.py` labelling the keyword gate as a Layer 3 scaffold decision (not model tool choice), naming the specific phrases it misses, and documenting the upgrade path to `get_booking_link` with `tool_choice="auto"`. Updated README and method.md to replace "agent-initiated booking" with "Python-gated scaffold routing."

Days 3–5: TBD
