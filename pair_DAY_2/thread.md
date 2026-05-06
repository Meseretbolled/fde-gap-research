# Day 2 — Tweet Thread

---

**Tweet 1** — Name the question
> My Week 10 sales agent had a booking trigger: if the prospect's reply contained "book," "schedule," or "availability," Python fetched Cal.com slots.
>
> A prospect who said "let's see what makes sense timing-wise" got nothing.
>
> I thought this was a model problem. It wasn't. 🧵

---

**Tweet 2** — The three layers
> When an agent picks the wrong tool (or skips one), three different things can cause it:
>
> Layer 1 — the model's own probability distribution
> Layer 2 — how you wrote the tool description
> Layer 3 — your scaffold forced or blocked the call
>
> They have three different fixes. Confusing them wastes days.

---

**Tweet 3** — The observable signal
> The signal that separates Layer 1 from everything else: finish_reason
>
> finish_reason: "tool_calls" → model committed to a structured tool call
> finish_reason: "stop" → model committed to plain text
>
> This is only meaningful when tool_choice="auto"
> If your scaffold used "required", the model didn't choose — you did.

---

**Tweet 4** — Description quality (Layer 2)
> Same task. Two tool descriptions:
>
> ❌ "Get company info" → finish_reason: stop, no tool called
> ✅ "Retrieve funding round, headcount, AI maturity — call before composing any email" → finish_reason: tool_calls
>
> The description is injected as plain text before your messages. It is prompt, not metadata.

---

**Tweet 5** — What about reasoning tokens?
> You might think: just log the model's reasoning to see why it chose a tool.
>
> Don't. Reasoning tokens:
> — require explicit opt-in (unavailable on most OpenRouter models)
> — may rationalize a decision already made, not cause it
> — can contain sensitive prospect data you haven't disclosed
>
> finish_reason + tool_choice_mode tells you everything you need.

---

**Tweet 6** — Link to blog
> Full explainer with A/B measurement script, a 3-layer trace schema, a drift detection function for multi-turn pipelines, and the MCP vs function calling distinction your questions probably need:
>
> https://medium.com/@meseretbolled/my-sales-agent-was-missing-booking-signals-and-it-was-never-the-models-fault-d22102622113
>
> Written for Gersum Asfaw and Hiwot Beyene as part of TRP1 Week 12 paired gap research.

---

#AgentInternals #FunctionCalling #LLMToolUse #TRP1
