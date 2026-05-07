# fde-gap-research

**Week 12 — Knowledge Gap Formulation for Compounding**
**Author:** Meseret Bolled
**Program:** TRP1 — Forward-Deployed Engineer Track
**Portfolio grounded in:** [tenacious-bench](https://github.com/Meseretbolled/Sales-Agent-Evaluation-Bench) (Week 11)

---

## What This Repo Is

Five days of paired gap research. Each day: one question I named from my own Week 11 work, one explainer I wrote for my partner's question, and one concrete edit back into my tenacious-bench portfolio.

By the end of the week: 5 gaps I named, 5 gaps I explained, 5 blog posts, 5 tweet threads, and 5 grounding commits to my existing work.

---

## Daily Pairs

| Day | Topic | My Question | Explainer I Wrote | Partner |
|-----|-------|------------|-------------------|---------|
| 1 | Evaluation & Statistics — LLM-as-judge biases | Does position bias in a single-response rubric judge inflate scores for criteria listed first in `TONE_JUDGE_PROMPT`? | LoRA merged vs unmerged inference serving | Gashaw Bekele |
| 2 | Agent and Tool-Use Internals | My 10-keyword booking gate has a systematic miss rate — would giving the model a `get_booking_link` tool schema catch the phrases my list misses? | Three layers of tool-use failure and how to tell them apart without reading the model's mind (Gersum Asfaw + Hiwot Beyene) | Gersum Asfaw, Hiwot Beyene |
| 3 | Training and Post-Training Mechanics | My DPO judge scores tone criteria in fixed order — does position bias in labels propagate through DPO training, and does my +0.1904 Delta A measure genuine improvement or Goodhart optimization? | SimPO per-token gradient decomposition: which tokens in near-identical pairs accumulate the most gradient under length normalization, and is the shipped critic token-localized or sequence-diffuse? | Kidane Gebremedhin |
| 4 | TBD | TBD | TBD | TBD |
| 5 | TBD | TBD | TBD | TBD |

---

## Daily Folders

| Folder | Contents |
|--------|----------|
| [pair_DAY_1/](pair_DAY_1/) | question.md · explainer.md · morning_call_summary.md · thread.md · evening_call_summary.md · signoff.md · grounding_commit.md · sources.md |
| [pair_DAY_2/](pair_DAY_2/) | Same structure — to be filled Day 2 |
| [pair_DAY_3/](pair_DAY_3/) | Same structure — to be filled Day 3 |
| [pair_DAY_4/](pair_DAY_4/) | Same structure — to be filled Day 4 |
| [pair_DAY_5/](pair_DAY_5/) | Same structure — to be filled Day 5 |

---

## Public Artifacts

### Blog Posts
- Day 1: [Merged vs Unmerged LoRA at Inference: When Are They Identical and When Do They Silently Diverge?](https://open.substack.com/pub/meseretbolled/p/merged-vs-unmerged-lora-at-inference?r=33718a&utm_campaign=post-expanded-share&utm_medium=web)
- Day 2: [My Sales Agent Was Missing Booking Signals — And It Was Never the Model's Fault](https://medium.com/@meseretbolled/my-sales-agent-was-missing-booking-signals-and-it-was-never-the-models-fault-d22102622113)
- Day 3: TBD — post in progress
- Day 4: TBD
- Day 5: TBD

### Tweet Threads
- Day 1: [Merged vs Unmerged LoRA at Inference](https://x.com/Meseret_Bolled/status/2051718207040323678?s=20)
- Day 2: TBD
- Day 3: TBD — thread drafted in [pair_DAY_3/thread.md](pair_DAY_3/thread.md)
- Day 4: TBD
- Day 5: TBD

---

## Final Deliverables
- [synthesis.md](synthesis.md) — 10 gaps closed, surprises, canonical reading list
- [canonical_list.md](canonical_list.md) — papers, tools, and patterns worth reading
- [portfolio_update.md](portfolio_update.md) — how this week improved my Week 10/11 work
