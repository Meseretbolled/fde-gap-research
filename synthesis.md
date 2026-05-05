# Week 12 Synthesis

**Author:** Meseret Bolled
**Submitted:** <!-- date -->

---

## The 10 Gaps Closed

### Gaps I Named (questions I asked, my partner explained)

| Day | My Question | Gap Closed? |
|-----|------------|-------------|
| 1 | Does fixed criterion ordering in my single-response rubric judge inflate scores for criteria listed first via position bias — and does that distortion touch my +25.4% lift? | Partially closed — mechanism understood, rotation audit not yet run |
| 2 | TBD | TBD |
| 3 | TBD | TBD |
| 4 | TBD | TBD |
| 5 | TBD | TBD |

### Gaps I Researched (my partner's questions, I explained)

| Day | Partner's Question | Their Sign-off |
|-----|-------------------|---------------|
| 1 | Are merged and unmerged LoRA weights mathematically guaranteed to produce identical logits, or are there conditions where they silently diverge? (Gashaw Bekele) | TBD — pending Gashaw's signoff |
| 2 | TBD | TBD |
| 3 | TBD | TBD |
| 4 | TBD | TBD |
| 5 | TBD | TBD |

---

## The Most Surprising Thing I Learned

**Day 1:** I had one defence for my tone judge — cross-family evaluation prevents self-preference bias — and I believed it was complete. The most surprising thing I learned is that self-preference bias and position bias are two entirely separate failure modes that require two entirely different mitigations. My cross-family defence says nothing about whether criteria listed first in the prompt receive inflated scores due to how autoregressive models generate tokens sequentially. I had been treating one mitigation as two, and I did not know they were different until today.

Days 2–5: TBD

---

## Canonical Reading List

See [canonical_list.md](canonical_list.md) for the full annotated list.

**Day 1 additions:**
- Hu et al. (2021) — LoRA original paper
- Wang et al. (2023) — LLM-as-judge position bias
- Zheng et al. (2023) — MT-Bench and Chatbot Arena judge calibration

---

## How This Week Changed My Work

**Day 1:** Added a named limitation paragraph to `tenacious-bench/methodology_rationale.md` distinguishing self-preference bias (mitigated) from position bias (unaudited). The +25.4% lift claim is now accompanied by an honest disclosure rather than an implied completeness.

Days 2–5: TBD
