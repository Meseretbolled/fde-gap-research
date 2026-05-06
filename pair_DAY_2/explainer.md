# Day 2 Explainer — Agent and Tool-Use Internals

**Written by:** Meseret Bolled
**Date:** May 6, 2026

---

## Part 1 — For Gersum Asfaw

# Three Layers of Tool-Use Failure — And How to Tell Them Apart Without Reading the Model's Mind

> **Gersum's question:** In my Tenacious workflow, how can I design MCP tool descriptions, function-calling traces, and multi-turn orchestration so I can distinguish: (1) model-level tool choice, (2) tool-selection effects caused by tool schema or description quality, and (3) scaffold-driven planning behavior across turns — without treating hidden reasoning tokens as part of the answer?

The short answer: these three causes are only distinguishable if you design your trace to capture them separately. The mechanism that makes it possible is `finish_reason` plus `tool_choice` mode plus a versioned tool description registry. Here is how each layer works and how to isolate it.

> **A note on MCP vs function calling:** Your question uses the term "MCP tool calls." MCP (Model Context Protocol) is a specific standard — originally from Anthropic — for connecting external tool servers to models via a structured protocol. Your actual implementation uses OpenRouter, which exposes the OpenAI-compatible `tools` parameter over a standard HTTP API. That is function calling, not MCP. The mechanics explained in this explainer apply directly to what you built. The distinction matters because a native MCP server adds a transport layer (the MCP host routes tool calls to registered servers) that the OpenAI-compatible API does not have. For your current pipeline, function calling is the right mental model and MCP is a future upgrade path if you want a standardised multi-tool server architecture.

---

### 1. Why One Failure Has Three Different Causes

When your agent calls the wrong tool, calls a tool with bad arguments, or skips a tool it should have called, one of three things caused it:

| Layer | What it means | Controlled by |
|---|---|---|
| **Layer 1 — Model tool choice** | The model's own probability distribution over tool names given the current context | Model weights + full message history |
| **Layer 2 — Description quality** | The tool name, description, and parameter fields shifted the model toward or away from selecting this tool | You, when you wrote the tool schema |
| **Layer 3 — Scaffold policy** | Your orchestration code forced, blocked, or pre-sequenced the tool call before the model made a free choice | Your pipeline code |

If you do not separate these, you will fix the wrong thing. A scaffold bug looks like a model bug. A bad description looks like a model reasoning failure. The three layers have three different fixes.

---

### 2. Layer 1 — What Model Tool Choice Is at the Token Level

When you call the OpenAI-compatible API with a `tools` parameter, the tool descriptions are injected as a structured block into the model's input. The model then generates either a `tool_calls` array (it wants to call a tool) or plain `content` text. The decision is observable as `finish_reason`.

**What the API call looks like:**

```python
response = client.chat.completions.create(
    model="deepseek/deepseek-chat",
    messages=[
        {"role": "system", "content": "You are a sales agent for Tenacious."},
        {"role": "user",   "content": "Stripe just posted 8 ML roles."}
    ],
    tools=[{
        "type": "function",
        "function": {
            "name": "enrich_company",
            "description": "Retrieve funding round, headcount, and AI maturity score for a company.",
            "parameters": {
                "type": "object",
                "properties": {
                    "company_name": {"type": "string", "description": "The exact company name"}
                },
                "required": ["company_name"]
            }
        }
    }],
    tool_choice="auto"   # model decides freely
)
```

**When the model chooses to call the tool:**

```python
choice = response.choices[0]
choice.finish_reason                               # "tool_calls"
choice.message.content                             # None
choice.message.tool_calls[0].function.name         # "enrich_company"
choice.message.tool_calls[0].function.arguments    # '{"company_name": "Stripe"}'
```

**When the model chooses NOT to call a tool:**

```python
choice.finish_reason      # "stop"
choice.message.content    # "Stripe posted 8 ML roles recently..."
choice.message.tool_calls # None
```

> **The mechanism:** `finish_reason: "tool_calls"` is the observable signal that the model committed to a structured tool invocation. `finish_reason: "stop"` means it committed to plain text. This is your Layer 1 signal — it is only valid when `tool_choice="auto"`, because only then did the model choose freely.

---

### 3. Layer 2 — How Tool Description Quality Affects Selection

The `description` field is not decorative. It is prompt text injected before your messages. A vague description reduces the probability that the model selects the right tool.

**Same task, two description versions:**

```python
# Version A — vague
{"name": "enrich_company", "description": "Get company info"}

# Version B — specific
{"name": "enrich_company", "description": (
    "Retrieve funding round type (Seed/A/B), raise amount, investor list, "
    "headcount, and AI maturity score (0–3) for a company by name. "
    "Call this before composing any outreach email."
)}
```

**A/B measurement script:**

```python
from openai import OpenAI

client  = OpenAI(base_url="https://openrouter.ai/api/v1", api_key="...")
message = {"role": "user", "content": "Stripe posted 8 ML roles. Series B in March."}
system  = {"role": "system", "content": "You are a sales agent for Tenacious."}

descriptions = {
    "v_vague":    "Get company info",
    "v_specific": ("Retrieve funding round, headcount, and AI maturity score "
                   "for a company by name. Call this before composing any email.")
}

for version, desc in descriptions.items():
    resp = client.chat.completions.create(
        model="deepseek/deepseek-chat",
        messages=[system, message],
        tools=[{"type": "function", "function": {
            "name": "enrich_company",
            "description": desc,
            "parameters": {"type": "object",
                           "properties": {"company_name": {"type": "string"}},
                           "required": ["company_name"]}
        }}],
        tool_choice="auto",
    )
    c    = resp.choices[0]
    tool = (c.message.tool_calls or [None])[0]
    print(f"{version}: finish={c.finish_reason}  tool={tool.function.name if tool else None}")
```

**Expected output:**

```
v_vague:    finish=stop        tool=None
v_specific: finish=tool_calls  tool=enrich_company
```

Run this across 15–20 representative prospect messages. A description that drops `tool_calls` rate by more than 20% compared to your best version is a Layer 2 problem — fix the description, not the model.

> **The rule:** Hold scaffold constant. Hold user message constant. Vary only the description. The delta in `finish_reason: "tool_calls"` rate is purely a description quality effect.

---

### 4. Layer 3 — Scaffold-Driven Behavior

`tool_choice` has four modes:

| `tool_choice` value | What it means | Layer |
|---|---|---|
| `"auto"` | Model decides freely | Layer 1 observable |
| `"required"` | Model must call a tool; it picks which one | Layer 3 — scaffold forced a call |
| `{"type": "function", "function": {"name": "enrich_company"}}` | Model must call this specific tool | Layer 3 — scaffold forced tool and name |
| `"none"` | Model cannot call any tool | Layer 3 — scaffold blocked all tools |

> **Common error:** Running your `enrich → compose → qualify → book → sync` pipeline with `tool_choice="required"` at each step and concluding "the model always enriches before composing." That is scaffold behavior, not model behavior. Switch to `tool_choice="auto"` and see what breaks. What breaks is the Layer 3 dependency you did not know you had.

Log `tool_choice_mode` at every turn. If it is anything other than `"auto"`, do not attribute the resulting tool selection to the model.

---

### 5. Why You Must Not Rely on Hidden Reasoning Tokens

Models with extended thinking (Claude's `<thinking>` blocks, OpenAI's `reasoning_effort`) emit internal reasoning before their response. Three reasons not to rely on these for debugging:

**1. They require explicit opt-in and are unavailable on most OpenRouter models.**

**2. They are not a reliable causal explanation.** A reasoning token saying "I'll call enrich_company" does not prove that reasoning caused the tool call. The token is generated sequentially and may rationalize a decision already made by earlier layers.

**3. They can contain sensitive prospect data.** If a trace captures "this person works at Stripe and their last funding was $X," logging it creates a data handling obligation not disclosed in your policy.

The correct alternative: design your trace to capture `finish_reason`, `tool_choice_mode`, `tool_selected`, and `description_version` at every turn. These are fully observable without reasoning tokens.

---

### 6. A Trace Schema That Separates All Three Layers

```python
import json

def log_turn(turn_index, response, tool_choice_mode, description_version, scaffold_forced=False):
    choice     = response.choices[0]
    tool_calls = choice.message.tool_calls or []
    return {
        "turn":                turn_index,
        "finish_reason":       choice.finish_reason,
        "tool_choice_mode":    tool_choice_mode,
        "model_chose_freely":  tool_choice_mode == "auto",
        "description_version": description_version,
        "scaffold_forced":     scaffold_forced,
        "tool_selected":       tool_calls[0].function.name if tool_calls else None,
        "arguments_valid":     _args_valid(tool_calls),
    }

def _args_valid(tool_calls):
    if not tool_calls:
        return None
    try:
        for tc in tool_calls:
            json.loads(tc.function.arguments)
        return True
    except Exception:
        return False
```

**Sample trace — 5-turn `enrich → compose → qualify → book → sync` run:**

```json
[
  {"turn": 1, "finish_reason": "tool_calls", "tool_choice_mode": "auto",
   "model_chose_freely": true,  "tool_selected": "enrich_company",   "scaffold_forced": false},
  {"turn": 2, "finish_reason": "tool_calls", "tool_choice_mode": "required",
   "model_chose_freely": false, "tool_selected": "compose_email",    "scaffold_forced": true},
  {"turn": 3, "finish_reason": "stop",       "tool_choice_mode": "auto",
   "model_chose_freely": true,  "tool_selected": null,               "scaffold_forced": false},
  {"turn": 4, "finish_reason": "tool_calls", "tool_choice_mode": "auto",
   "model_chose_freely": true,  "tool_selected": "get_booking_link", "scaffold_forced": false},
  {"turn": 5, "finish_reason": "tool_calls", "tool_choice_mode": "required",
   "model_chose_freely": false, "tool_selected": "sync_to_hubspot",  "scaffold_forced": true}
]
```

- **Turns 2 and 5** — `scaffold_forced: true`. Do not attribute to the model.
- **Turn 3** — model freely chose not to call a tool. Worth investigating: vague description or missing context?
- **Turns 1 and 4** — genuine model tool choice.

---

### 7. Attribution Decision Tree

```
Agent called wrong tool or skipped a required tool
│
├── tool_choice_mode ≠ "auto" at that turn?
│    └── YES → Layer 3 (scaffold). Fix: change tool_choice or pipeline ordering.
│
├── tool_choice_mode was "auto" but a different description version was live?
│    └── YES → Layer 2 (description). Fix: rewrite the description.
│
└── Same description, same scaffold, "auto" mode, still wrong?
     └── Layer 1 (model). Fix: add a worked example to the description
         or provide richer context in the user message for that turn.
```

---

### Sources

1. **OpenAI. (2024). Function Calling.** OpenAI API Reference.
   https://platform.openai.com/docs/guides/function-calling
   *Canonical reference for `tools`, `finish_reason`, and all `tool_choice` modes.*

2. **Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., & Cao, Y. (2023). ReAct: Synergizing Reasoning and Acting in Language Models.** ICLR 2023.
   https://arxiv.org/abs/2210.03629
   *Establishes the Reason-Act loop and the conceptual basis for separating model reasoning from scaffold execution — essential for Layer 3 isolation.*

---

---

## Part 2 — For Hiwot Beyene

# What the Model Actually Sees Before It Picks a Tool — And How to Catch Drift Across Turns

> **Hiwot's question:** In my Week 10 Conversion Engine, when the model selects tools across `enrich -> compose -> send -> reply -> book`, how can I design and trace MCP tool calls so I can clearly separate: (1) true model tool-choice behavior at token level, (2) tool-routing quality caused by tool descriptions, and (3) scaffold-policy effects in multi-turn planning — while logging only safe, useful outputs (not hidden reasoning traces)?

Hiwot's question adds two things Gersum's does not: she wants to understand how the model *represents* tools in-token before selecting one, and she specifically asks about multi-turn planning drift — how tool selection degrades as conversation history grows. Both have concrete, actionable answers.

> **A note on MCP vs function calling:** Your question uses the term "MCP tool calls." MCP (Model Context Protocol) is a structured standard for connecting external tool servers to a model host — the host routes tool calls to registered MCP servers over a defined transport layer. Your Week 10 implementation uses OpenRouter's OpenAI-compatible API with a `tools` parameter, which is function calling — not MCP. Everything in this explainer applies directly to your code. If you later want to standardise your tool layer across multiple agents or expose your tools to different models without rewriting schemas each time, MCP is the upgrade path. For now, function calling is what you have and what matters.

---

### 1. How the Model Represents Tools Before Selection

When you pass `tools=[...]` to the API, the model does not receive a Python dict. The tool schemas are serialized into a special context block inserted before your system message. It looks roughly like this in the model's raw input:

```
# Tools

## enrich_company
Retrieve funding round, headcount, and AI maturity score for a company by name.

### Parameters
- company_name (string, required): The exact company name

## compose_email
Write a cold outreach email using a hiring brief and ICP segment.

### Parameters
- prospect_name (string, required): Prospect's first name
- segment (integer, required): ICP segment number (1–4)
- signal_summary (string, required): One-sentence verified signal
```

The model reads this block as plain text before your messages. This is why the description is so load-bearing — the model uses those words to assign probability to each tool name before it has even seen your user message for this turn.

**What this means practically:** A tool named `enrich_company` with description "Get company info" teaches the model almost nothing about when to use it. A tool named `enrich_company` with description "Retrieve funding round, headcount, and AI maturity — call this at turn 1 of any new prospect before composing" gives the model a timing instruction, a trigger condition, and a scope. That is three times more signal.

---

### 2. Multi-Turn Planning Drift — Where It Comes From

In a 5-turn pipeline (`enrich → compose → send → reply → book`), the message history grows with every turn. By turn 4, the model's context contains:

- The original system prompt
- The full tool schema block
- All previous user messages
- All previous assistant messages (including tool call arguments and tool results)

This creates two drift risks:

**Risk 1 — Context dilution.** The tool schema block, written once at the start, is now buried under 3–4 turns of conversation. The model's attention to the description weakens as the history grows. A tool that was reliably selected at turn 1 may be skipped at turn 4 because the description is effectively further away from the current query.

**Risk 2 — History-induced constraint.** If the model sees that it already called `enrich_company` in the history, it will be less likely to call it again — even if re-enrichment would be appropriate (e.g., a different company name was mentioned). The history acts as implicit context that changes selection probability.

**How to detect drift in your traces:**

```python
def detect_drift(trace_log):
    """
    Check whether tool selection rate drops across turns for the same tool.
    A drop of >20pp between turn 1 and turn 4 for the same tool signals drift.
    """
    from collections import defaultdict
    by_turn = defaultdict(list)
    for record in trace_log:
        by_turn[record["turn"]].append(record["tool_selected"])

    for turn, tools in sorted(by_turn.items()):
        selected = [t for t in tools if t is not None]
        rate = len(selected) / len(tools) if tools else 0
        print(f"Turn {turn}: tool_calls rate = {rate:.0%}  tools used: {set(selected)}")
```

If turn 1 shows 90% tool_calls rate and turn 4 shows 40%, that is drift. The next question is whether it is Layer 2 (description lost relevance by turn 4) or Layer 1 (model reasoning shifted). You test this the same way as before: hold the history constant, vary the description version, and measure the delta.

---

### 3. Hidden Scratch Work vs Auditable Output

Hiwot's question names a distinction that matters for production systems: some model output is internal scratch work and some is auditable output that should be logged and defensible.

Here is the clean separation:

| Type | What it is | Should you log it? |
|---|---|---|
| **Tool call arguments** | Structured JSON the model emitted to call a tool | Yes — always. This is auditable output. |
| **Tool result** | What your function returned to the model | Yes — always. This is ground truth for the next turn. |
| **Assistant response text** | The model's final reply to the prospect | Yes — always. This is what the prospect sees. |
| **Reasoning tokens** (`<thinking>`) | Internal scratch work before the response | No — unless you have explicit consent and a legal basis. Do not log. |
| **`finish_reason`** | Whether the model chose a tool or text | Yes — this is metadata, not content, and is safe to log. |

The key insight: you do not need reasoning tokens to debug tool failures. Everything you need to separate the three layers is in `finish_reason`, `tool_calls`, tool results, and the `tool_choice_mode` you set. Reasoning tokens add cost, add privacy risk, and add unreliable signal. Leave them off.

**Safe, complete trace record for one turn:**

```python
def safe_log_turn(turn_index, response, tool_choice_mode, description_version,
                  tool_result=None, scaffold_forced=False):
    choice     = response.choices[0]
    tool_calls = choice.message.tool_calls or []
    return {
        "turn":                turn_index,

        # What the model decided
        "finish_reason":       choice.finish_reason,
        "tool_choice_mode":    tool_choice_mode,
        "model_chose_freely":  tool_choice_mode == "auto",
        "tool_selected":       tool_calls[0].function.name if tool_calls else None,
        "tool_arguments":      tool_calls[0].function.arguments if tool_calls else None,

        # What came back
        "tool_result_summary": str(tool_result)[:200] if tool_result else None,

        # Attribution helpers
        "description_version": description_version,
        "scaffold_forced":     scaffold_forced,

        # Deliberately NOT included: reasoning tokens, full prospect data
    }
```

This log is safe to store, safe to share with a tutor or hiring manager, and contains everything needed to run the three-layer attribution analysis.

---

### 4. Applying This to Hiwot's `enrich → compose → send → reply → book` Pipeline

| Turn | Tool | `tool_choice` to use | What to watch |
|---|---|---|---|
| 1 — enrich | `enrich_company` | `"auto"` | Does model freely select? If not, description is the problem. |
| 2 — compose | `compose_email` | `"auto"` | Does history from turn 1 help or hurt selection? |
| 3 — send | `send_outreach_email` | `"required"` | Scaffold-forced — log `scaffold_forced: true`, do not attribute to model. |
| 4 — reply | `handle_reply` | `"auto"` | Most likely turn for drift — 3 prior turns dilute the schema. Check turn 4 selection rate. |
| 5 — book | `get_booking_link` | `"auto"` | Test whether intent keywords in the reply drive model selection or whether description does the work. |

Turn 4 is the highest-risk turn for drift in a 5-step pipeline. If your booking conversion fails, start there.

---

### Sources

1. **OpenAI. (2024). Function Calling.** OpenAI API Reference.
   https://platform.openai.com/docs/guides/function-calling
   *Canonical reference for tool schema serialization, `finish_reason`, and `tool_choice` modes. Section on parallel function calls is relevant for multi-step pipelines.*

2. **Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., & Cao, Y. (2023). ReAct: Synergizing Reasoning and Acting in Language Models.** ICLR 2023.
   https://arxiv.org/abs/2210.03629
   *The foundational paper for multi-turn reasoning-action loops. Section 3 explains why separating "what the model chose" from "what the scaffold forced" is necessary for reliable agent debugging — directly applicable to Hiwot's drift analysis.*
