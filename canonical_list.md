# Canonical List — Week 12

**Author:** Meseret Bolled
**For:** TRP1 Cohort Canon

---

## Papers

| Title | Authors | Link | Why It Matters |
|-------|---------|------|----------------|
| LoRA: Low-Rank Adaptation of Large Language Models | Hu et al. (2021) | https://arxiv.org/abs/2106.09685 | Primary source for merged vs unmerged forward-pass arithmetic and the α/r scaling factor. Read before any LoRA deployment decision. |
| Large Language Models are not Fair Evaluators | Wang et al. (2023) | https://arxiv.org/abs/2305.17926 | Demonstrates primacy and recency effects in rubric-style LLM evaluation. Proposes the rotation-and-average mitigation for position bias. |
| Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena | Zheng et al. (2023) | https://arxiv.org/abs/2306.05685 | Establishes position bias mechanism in pairwise judges. Section 4.2 is the canonical reference for why fixed ordering inflates scores. |
| ReAct: Synergizing Reasoning and Acting in Language Models | Yao et al. (2023) | https://arxiv.org/abs/2210.03629 | Foundational paper for multi-turn reasoning-action loops. Section 3 is the basis for separating model tool choice from scaffold-driven execution — essential reading before debugging any multi-turn agent. |
| Toolformer: Language Models Can Teach Themselves to Use Tools | Schick et al. (2023) | https://arxiv.org/abs/2302.04761 | Explains how tool-use capability is acquired during training via conditional generation. Directly explains why tool description quality affects selection probability — the model learned from descriptive schemas. |
| Direct Preference Optimization: Your Language Model is Secretly a Reward Model | Rafailov et al. (2023) | https://arxiv.org/abs/2305.18290 | Primary source for DPO's loss formulation and implicit reward derivation (Section 4). The key insight: DPO has no representation of what chosen/rejected are — only their labels. Label bias propagates as faithfully as label signal. |
| SimPO: Simple Preference Optimization with a Reference-Free Reward | Meng et al. (2024) | https://arxiv.org/abs/2405.14734 | Primary source for the length-normalized reward R(y) = (1/|y|) × Σ log π(y_t). The 1/|y| per-token weight is load-bearing for gradient smear analysis — each token contributes equally regardless of position. |
| Scaling Laws for Reward Model Overoptimization | Gao et al. (2023) | https://arxiv.org/abs/2210.10760 | Canonical empirical evidence that proxy reward keeps rising while gold reward plateaus. Figure 1 is the standard illustration of the Goodhart gap in preference learning. Read before reporting any proxy-eval lift as a training success. |

---

## Tools & Patterns

| Tool / Pattern | What It Does | Link | When to Use It |
|---------------|-------------|------|----------------|
| PEFT `merge_and_unload()` | Fuses LoRA B×A matrices permanently into base weights, eliminating the adapter as a separate module | https://huggingface.co/docs/peft/conceptual_guides/lora | Use before production serving of a stable single adapter. Always call `model.eval()` first. |
| Criterion rotation audit | Run rubric judge twice with original and reversed criterion order, compare per-criterion pass rates | — | Use before reporting rubric judge scores as bias-free. Flag any criterion with > 8 pp gap between runs. |
| OpenAI Function Calling API | Exposes `tools`, `tool_choice`, and `finish_reason` for model-driven tool selection | https://platform.openai.com/docs/guides/function-calling | Use any time you need the model to decide when to call an external API. Set `tool_choice="auto"` and log `finish_reason` to observe free model choice. |
| Three-layer tool attribution (Layer 1 / 2 / 3) | Framework for separating model tool choice, description quality effects, and scaffold policy effects in agent traces | — | Use before debugging any tool-selection failure. Attribute to the right layer before attempting a fix. |
| Criterion-rotation agreement audit | Re-run rubric judge N× per pair with different criterion orderings; compare majority-vote label to original fixed-order label. Agreement rate < 0.90 = material bias present | — | Use before reporting any rubric-scored lift as genuine. A 40-pair sample with 5 orderings is sufficient to detect material position bias. |
| Mask-and-rescore probe (SimPO) | Mask discriminative token spans in preference pairs; rescore reward gap before and after masking; compute survival ratio | — | Use to determine whether a shipped critic is token-localized (survival < 20%, safe for novel body shapes) or sequence-diffuse (survival > 50%, will mis-fire on format changes). |
| Dual eval (proxy vs gold Delta A) | Score held-out outputs with fixed-order rubric AND rotation-averaged rubric; gap between the two Delta A values = Goodhart tax in real units | — | Use to put a concrete number on how much of a reported lift is proxy-reward inflation rather than genuine quality improvement. |

---

## Engineering Patterns Worth Knowing

- **Always call `model.eval()` before LoRA unmerged inference.** Forgetting this leaves dropout active on the A branch, making the adapter partially invisible. No error is raised. This is the most common silent failure for a deployed adapter that "appears to have no effect."

- **Cross-family evaluation and position bias are independent.** Using a different model family as judge prevents self-preference bias only. It does nothing about criterion ordering effects in a rubric judge. Treat them as separate checklists.

- **Disclose known limitations explicitly.** A benchmark methodology that names what it does not protect against is more defensible than one that implies completeness. Name the bias, quantify the estimated impact, and move on.

- **A keyword gate is a scaffold decision, not model tool choice.** If Python decides whether to fetch a tool result before the LLM is called, the model never had the option. The miss rate is architectural. The fix is to pass a tool schema with `tool_choice="auto"` and let `finish_reason` tell you whether the model chose to call it.

- **MCP and function calling are not the same thing.** MCP (Model Context Protocol) is a structured transport standard for connecting tool servers to model hosts. The OpenAI-compatible `tools` parameter is function calling over HTTP. Most Week 10 implementations use function calling, not MCP — even if files are named `*_mcp.py`.

- **DPO is a label-faithful transducer.** The DPO loss has no representation of what chosen and rejected *are* — only of their labels. A consistent direction in labeling errors (e.g., position bias inflating first-listed criteria) becomes a gradient direction DPO learns as faithfully as genuine signal. Bias is not filtered; it is absorbed. Audit labels before training, not after.

- **The Goodhart gap has to be measured in real units.** Reporting a proxy Delta A (from a fixed-order rubric) without a rotation-averaged gold Delta A leaves the Goodhart tax unnamed and unquantified. Run dual eval on held-out pairs and report both numbers.

- **Length normalization in SimPO changes who accumulates gradient.** The 1/|y| weight makes every token contribute equally. On near-identical pairs, pre-divergence tokens cancel; tokens after the discriminative position accumulate gradient from context divergence, not quality difference. Know which critic you shipped before expanding to new body shapes.
