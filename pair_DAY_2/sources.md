# Day 2 — Sources

## Canonical Papers / Primary Sources

1. **Function Calling**
   - Authors: OpenAI
   - Link: https://platform.openai.com/docs/guides/function-calling
   - Why I used it: This is the canonical reference for the `tools` parameter, `finish_reason: "tool_calls"`, and all four `tool_choice` modes (`"auto"`, `"required"`, forced name, `"none"`). It is the primary source for Layer 1 (model free choice) and Layer 3 (scaffold policy) in the explainer. All code examples derive from the request/response format documented here.

2. **ReAct: Synergizing Reasoning and Acting in Language Models**
   - Authors: Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., & Cao, Y.
   - Link: https://arxiv.org/abs/2210.03629
   - Why I used it: ReAct establishes the Reason-Act loop that underlies all multi-turn tool-using agents and provides the conceptual basis for separating "what the model chose" from "what the scaffold executed." Section 3 is directly cited as the justification for Layer 3 isolation and the multi-turn drift analysis in Hiwot's section.

---

## Tool or Pattern Used

- **Tool/Pattern:** OpenAI-compatible `tools` API via OpenRouter
- **What I did with it:** Wrote and ran the A/B description quality measurement script comparing `v_vague` vs `v_specific` descriptions for the `enrich_company` tool across 15 representative prospect messages.
- **What I observed:** With a vague description ("Get company info"), `finish_reason` returned `"stop"` and no tool was selected. With a specific description naming the fields, trigger condition, and scope, `finish_reason` returned `"tool_calls"` with `enrich_company` selected. This confirmed that description quality is a measurable, independent variable — not a model capability question.

---

## Follow-on Reading

- Schick, T. et al. (2023). Toolformer: Language Models Can Teach Themselves to Use Tools. https://arxiv.org/abs/2302.04761 — for understanding how tool-use capability is acquired during training, which explains why description quality matters (the model learned to use tools from examples that had descriptive schemas).
