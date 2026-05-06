# My Sales Agent Was Missing Booking Signals — And It Was Never the Model's Fault

**By Meseret Bolled · Week 12, TRP1 Forward-Deployed Engineer Track**

---

I built a sales agent for Tenacious Consulting. It reads prospect replies, understands context, and offers a Cal.com booking link when the prospect is ready to talk.

At least, that was the idea.

In practice, a prospect who replied *"let's see what makes sense timing-wise"* got a generic follow-up email with no booking link. A prospect who replied *"I'd be open to a conversation"* got nothing. Meanwhile, a prospect who replied *"I don't have time for this"* — clearly not interested — sometimes got a booking offer anyway, because the word **time** matched my keyword list.

For weeks I assumed this was a model problem. The model was "failing to detect intent." I kept tweaking prompts.

It was not a model problem. The model never had a chance.

---

## The Actual Architecture

Here is what my `conversation_manager.py` actually did on every prospect reply:

```python
wants_booking = any(kw in reply_text.lower() for kw in [
    "book", "schedule", "call", "meet", "time", "availability",
    "calendar", "30 min", "happy to", "interested"
])

if wants_booking:
    slots = get_available_slots()   # fetch Cal.com
    # inject slot times into context

# then call the LLM
text, usage = chat(messages=history, system=SYSTEM + context)
```

Python runs a substring check. If any of 10 fixed words appear, it fetches Cal.com slots and pastes them into the prompt. Then — and only then — the LLM is called.

If no keyword matches, the model receives a prompt with no mention that booking is even possible. It is not "deciding not to book." It was never offered the option.

I had described this in my README as *"agent-initiated booking."* That was wrong. Python initiated everything. The agent was downstream.

---

## Two Scaffold Policies, Not Two Model Capabilities

This is the key distinction my peers Gersum Asfaw and Hiwot Beyene helped me name: I was not comparing a dumb approach to a smart one. I was comparing **two scaffold policies** for the same underlying model.

| Policy | What happens | Where the decision lives |
|---|---|---|
| **A — Keyword gate + injection** | If a keyword matches, Python fetches slots and pastes them into the user message before one `chat()` call | Python, before inference |
| **B — Tool calling** | The model receives a `get_booking_link` tool schema, may emit a `tool_calls` payload, runtime executes it, result returns as a `tool` role message | The model, during inference |

The miss rate I was seeing was not a recall failure of the model. It was a recall ceiling of the keyword list. The model's ability to detect intent was never measured — because intent detection never happened inside the model.

---

## What Actually Changes at the Token Level

This was the question I asked: *what exactly changes at the token level when you give the model a tool schema?*

The answer is structural. Here is the message transcript for each path on the same prospect reply: *"I'd be open to a conversation."*

**Path A — keyword gate, no match (the miss):**

```
system:  You are a sales assistant for Tenacious…
user:    Prospect replied: "I'd be open to a conversation."
         [no calendar block — Python decided not to fetch it]
assistant: <model generates a reply with no booking option available>
```

The model never sees slots. There is no path to a booking link. The conversation continues without one.

**Path B — tool calling:**

Request 1 — includes `tools: [{ name: "get_booking_link", description: "Call when the prospect expresses scheduling interest…" }]`

```
assistant: tool_calls = [{ name: "get_booking_link", arguments: {} }]
```

The model read *"I'd be open to a conversation"* and inferred scheduling intent from meaning, not substrings. It emitted a structured request. The runtime fetched Cal.com slots and sent them back:

Request 2:
```
tool:      name=get_booking_link, content='{"slots": ["Tue 10:00", "Wed 14:00"]}'
assistant: "Happy to — here are a couple of times that could work…"
```

The difference is not about the model being smarter. It is about **what the model is allowed to predict**. In Path A, Continuation B — a structured tool call — does not exist in the context window. In Path B, it does. The model chooses between plain assistant text and a `tool_calls` payload based on the meaning it infers from the reply.

As Gersum put it: *the architecture changes what the model is allowed to predict.*

---

## The Three Layers I Was Confusing

Before this week I had no framework for debugging tool failures in agents. Every missed booking looked the same. My peers introduced a three-layer attribution model that I now use for everything:

**Layer 1 — Model tool choice**
The model's own probability distribution over available actions, given the current context. Observable via `finish_reason: "tool_calls"` vs `"stop"` when `tool_choice="auto"`.

**Layer 2 — Description quality**
How you wrote the tool name, description, and parameters shifts the model's probability toward or away from selecting that tool. This is prompt, not magic. A vague description like *"Get booking info"* will underperform a specific one that names when to call and what it returns.

**Layer 3 — Scaffold policy**
Whether your orchestration code forced, blocked, or pre-decided the tool call before the model had a chance. If `tool_choice="required"`, the model did not choose. If Python ran a keyword gate, the model did not choose. Everything you attributed to the model in those cases was actually your scaffold.

My keyword gate was a **Layer 3 problem** that I had been treating as a **Layer 1 problem**. No amount of prompt tuning was going to fix it.

---

## What Tool Calling Does Not Fix Automatically

Hiwot was honest about the engineering reality, and I want to be too.

**It is not a drop-in swap.** Native function calling in the OpenAI-compatible API typically requires two model forward passes: the model emits `tool_calls`, your runtime executes the tool, then you send the result back for a second completion. That is more latency and slightly more cost per reply than a single `chat()` call. You are building a small state machine, not swapping one line.

**Vague tool descriptions cause false positives.** If the description is broad (*"Call whenever the prospect seems open"*), the model will over-call and offer booking links to replies that are not ready. The description must be tight — naming when to call *and* when not to.

**Your keyword false positive problem does not disappear.** My gate was firing on *"I don't have time for this"* because **time** matched. Tool calling solves the miss rate problem; it does not automatically solve precision unless your tool description explicitly tells the model when *not* to invoke it.

The recommended production design is **hybrid**:
1. Keep the keyword gate as a high-precision fast path for unambiguous signals
2. Expose `get_booking_link` as a semantic fallback for non-matching but engaged replies
3. Measure recall lift and false positive rate separately before a pilot

---

## The Fix I Made to My Codebase

I did not rewrite the booking flow this week — a two-step LLM loop is a real architecture change that belongs in a proper sprint.

What I did do was fix the documentation that was misrepresenting the architecture.

In `conversation_manager.py`, above the keyword gate:

```python
# SCAFFOLD DECISION (Layer 3) — not model tool choice.
# Python decides whether to fetch Cal.com slots before the LLM is called.
# The model never receives a tool schema and never emits a tool_call for booking.
# Known miss rate: "let's see what makes sense timing-wise",
# "I'd be open to a conversation", "what does your process look like"
# Known false positive: "I don't have time for this" matches "time"
# Upgrade path: pass get_booking_link schema with tool_choice="auto"
# and let finish_reason determine whether the model chose to book.
wants_booking = any(kw in reply_text.lower() for kw in [...])
```

In `README.md`, I replaced *"agent-initiated booking"* with *"Python-gated scaffold routing."*

A hiring manager or client engineer reading the architecture now sees what the system actually does — and a named path to making it better.

---

## The Bigger Lesson

I had a file called `hubspot_mcp.py`. I described my system as having "agentic tool use." Neither of those things was true in the way I meant them.

MCP (Model Context Protocol) is a specific standard — the model host routes tool calls to registered MCP servers. What I had was a HubSpot Python SDK wrapper called after the LLM generated text. The model never emitted a function call. Python called HubSpot.

This is not a failure. It is a completely valid architecture for many use cases. The problem was the description — calling it something it was not.

The lesson I'm taking into Week 13: **describe your system accurately before you optimize it.** You cannot debug a Layer 3 scaffold problem if you believe it is a Layer 1 model problem. The label you put on a component changes what you try to fix.

---

## Sources

1. **OpenAI. Function Calling.** https://platform.openai.com/docs/guides/function-calling
   The canonical reference for `tools`, `finish_reason`, and `tool_choice` modes.

2. **Yao et al. (2023). ReAct: Synergizing Reasoning and Acting in Language Models.** ICLR 2023. https://arxiv.org/abs/2210.03629
   The foundational paper for multi-turn reasoning-action loops and the basis for separating model choice from scaffold execution.

3. **Schick et al. (2023). Toolformer: Language Models Can Teach Themselves to Use Tools.** https://arxiv.org/abs/2302.04761
   Explains how tool-use capability is acquired during training — and why description quality directly affects selection probability.

---

*Written as part of TRP1 Week 12 paired gap research. Thanks to Gersum Asfaw and Hiwot Beyene for the explainers that closed this gap.*
