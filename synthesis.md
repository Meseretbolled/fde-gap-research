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
| 3 | My DPO judge scores tone criteria in a fixed order. Position bias inflates scores for criteria listed first. If 20% of my 159 preference labels were nudged by ordering effects, does my +0.1904 Delta A reflect genuine outreach improvement — or did I measure bias twice? | Closed — Kidane's explainer named the DPO-as-label-faithful-transducer mechanism and the criterion-rotation agreement audit |
| 4 | TBD | TBD |
| 5 | TBD | TBD |

### Gaps I Researched (my partner's questions, I explained)

| Day | Partner's Question | Their Sign-off |
|-----|-------------------|---------------|
| 1 | Are merged and unmerged LoRA weights mathematically guaranteed to produce identical logits, or are there conditions where they silently diverge? (Gashaw Bekele) | TBD — pending Gashaw's signoff |
| 2 | How to distinguish model tool choice, description quality effects, and scaffold policy in multi-turn traces without relying on hidden reasoning tokens (Gersum Asfaw + Hiwot Beyene) | Closed — both confirmed mechanism and trace schema landed |
| 3 | Under SimPO's length-normalized loss, which tokens in near-identical preference pairs accumulate the most gradient — and does a shipped critic turn into a context-diffuse scorer or a token-localized tone detector? (Kidane Gebremedhin) | Closed — Kidane confirmed survival ratio thresholds and the asymmetric-in-time gradient framing |
| 4 | TBD | TBD |
| 5 | TBD | TBD |

---

## The Most Surprising Thing I Learned

**Day 1:** I had one defence for my tone judge — cross-family evaluation prevents self-preference bias — and I believed it was complete. The most surprising thing I learned is that self-preference bias and position bias are two entirely separate failure modes that require two entirely different mitigations. My cross-family defence says nothing about whether criteria listed first in the prompt receive inflated scores due to how autoregressive models generate tokens sequentially. I had been treating one mitigation as two, and I did not know they were different until today.

**Day 2:** I thought my keyword gate was a model problem — the model was "failing to detect intent." Gersum's explainer showed the miss happens before inference. The model never receives a tool schema so Continuation B (a tool call) does not exist as a token path. The gap is architectural, not a model capability issue. The surprising thing is how confidently I had described the booking flow as "agent-initiated" when Python was making every decision.

**Day 3:** I reported +0.1904 Delta A and treated it as evidence that DPO training improved the agent. I had noted the judge uses a fixed criterion order as a limitation but I did not understand why it mattered for training. Kidane's explainer showed that DPO has no representation of what chosen and rejected *are* — only of their labels. So any consistent direction in the labeling errors becomes a gradient direction DPO learns faithfully. I had been thinking of the bias as an evaluation problem; it is a training data problem. The Goodhart framing — proxy reward rises, gold reward may plateau — named the gap between what my Delta A measures and what I wanted it to measure. I did not have a name for that gap before.

---

## Canonical Reading List

See [canonical_list.md](canonical_list.md) for the full annotated list.

**Day 1 additions:**
- Hu et al. (2021) — LoRA original paper
- Wang et al. (2023) — LLM-as-judge position bias
- Zheng et al. (2023) — MT-Bench and Chatbot Arena judge calibration

**Day 2 blog post:** [My Sales Agent Was Missing Booking Signals — And It Was Never the Model's Fault](https://medium.com/@meseretbolled/my-sales-agent-was-missing-booking-signals-and-it-was-never-the-models-fault-d22102622113)

**Day 2 additions:**
- OpenAI Function Calling docs — canonical reference for `finish_reason`, `tool_choice` modes, and tool schema format
- Yao et al. (2023) ReAct — reasoning-action loop and the basis for separating model choice from scaffold execution
- Schick et al. (2023) Toolformer — how tool-use is learned, explains why description quality affects selection probability

**Day 3 blog post:** TBD

**Day 3 additions:**
- Rafailov et al. (2023) DPO — primary source for the loss formulation, implicit reward derivation, and per-token gradient structure showing DPO is a label-faithful transducer
- Meng et al. (2024) SimPO — primary source for length-normalized reward; per-token 1/|y| gradient weight is derived directly from Equation 3
- Gao et al. (2023) Scaling Laws for Reward Model Overoptimization — canonical evidence that proxy reward keeps rising while gold reward plateaus (Figure 1); names the Goodhart gap in preference learning

---

## How This Week Changed My Work

**Day 1:** Added a named limitation paragraph to `tenacious-bench/methodology_rationale.md` distinguishing self-preference bias (mitigated) from position bias (unaudited). The +25.4% lift claim is now accompanied by an honest disclosure rather than an implied completeness.

**Day 2:** Added an architectural comment to `conversion-engine/agent/agent_core/conversation_manager.py` labelling the keyword gate as a Layer 3 scaffold decision (not model tool choice), naming the specific phrases it misses, and documenting the upgrade path to `get_booking_link` with `tool_choice="auto"`. Updated README and method.md to replace "agent-initiated booking" with "Python-gated scaffold routing."

**Day 3:** Added a named limitation paragraph to `tenacious-bench/methodology_rationale.md` directly below the +0.1904 Delta A figure. The paragraph names the tone compliance judge's fixed-order position bias, states the criterion-rotation agreement audit has not yet been run, and reads the Delta A as an upper bound on genuine quality improvement with an unknown Goodhart tax. It specifies the exact audit (40-pair sample, 5 orderings, majority-vote comparison) and the interpretation thresholds (≥0.90 = defensible with caveat, 0.70–0.90 = material bias, <0.70 = retrain with rotation). The headline number is unchanged; the honest disclosure around it is new.

Days 4–5: TBD
