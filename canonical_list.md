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

---

## Tools & Patterns

| Tool / Pattern | What It Does | Link | When to Use It |
|---------------|-------------|------|----------------|
| PEFT `merge_and_unload()` | Fuses LoRA B×A matrices permanently into base weights, eliminating the adapter as a separate module | https://huggingface.co/docs/peft/conceptual_guides/lora | Use before production serving of a stable single adapter. Always call `model.eval()` first. |
| Criterion rotation audit | Run rubric judge twice with original and reversed criterion order, compare per-criterion pass rates | — | Use before reporting rubric judge scores as bias-free. Flag any criterion with > 8 pp gap between runs. |
| OpenAI Function Calling API | Exposes `tools`, `tool_choice`, and `finish_reason` for model-driven tool selection | https://platform.openai.com/docs/guides/function-calling | Use any time you need the model to decide when to call an external API. Set `tool_choice="auto"` and log `finish_reason` to observe free model choice. |
| Three-layer tool attribution (Layer 1 / 2 / 3) | Framework for separating model tool choice, description quality effects, and scaffold policy effects in agent traces | — | Use before debugging any tool-selection failure. Attribute to the right layer before attempting a fix. |

---

## Engineering Patterns Worth Knowing

- **Always call `model.eval()` before LoRA unmerged inference.** Forgetting this leaves dropout active on the A branch, making the adapter partially invisible. No error is raised. This is the most common silent failure for a deployed adapter that "appears to have no effect."

- **Cross-family evaluation and position bias are independent.** Using a different model family as judge prevents self-preference bias only. It does nothing about criterion ordering effects in a rubric judge. Treat them as separate checklists.

- **Disclose known limitations explicitly.** A benchmark methodology that names what it does not protect against is more defensible than one that implies completeness. Name the bias, quantify the estimated impact, and move on.

- **A keyword gate is a scaffold decision, not model tool choice.** If Python decides whether to fetch a tool result before the LLM is called, the model never had the option. The miss rate is architectural. The fix is to pass a tool schema with `tool_choice="auto"` and let `finish_reason` tell you whether the model chose to call it.

- **MCP and function calling are not the same thing.** MCP (Model Context Protocol) is a structured transport standard for connecting tool servers to model hosts. The OpenAI-compatible `tools` parameter is function calling over HTTP. Most Week 10 implementations use function calling, not MCP — even if files are named `*_mcp.py`.
