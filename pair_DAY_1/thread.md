# Day 1 — Tweet Thread

**Topic:** LoRA merged vs unmerged inference — when are they identical and when do they silently diverge?

---

**Tweet 1** — Name the question
> You trained a LoRA adapter. Loss dropped from 3.08 → 0.42.
> But your rubric scores didn't move at all.
>
> Before blaming the backbone — did you check whether your adapter
> is actually being applied at inference?
>
> Merged vs unmerged LoRA are NOT always identical. Here's why 🧵

---

**Tweet 2** — Load-bearing mechanism (the math)
> LoRA adds two matrices B and A to a frozen weight W.
>
> Unmerged: h = W·x + (α/r)·B·A·x  (computed live every forward pass)
> Merged:   h = W'·x  where W' = W + (α/r)·B·A  (fused once, offline)
>
> Algebraically identical. But 3 real-world conditions break this.

---

**Tweet 3** — The 3 divergence conditions
> 1. fp16 rounding — two separate matmuls accumulate error differently.
>    Gap is usually < 1e-3. Negligible for r=16 on a 0.5B model.
>
> 2. α/r scaling bug — if the library applies the scale in merge but
>    not in dynamic inference (or vice versa), you get a silent wrong answer.
>
> 3. Dropout left ON — the most common silent failure. If you forget
>    model.eval(), the A branch randomly zeroes activations. Adapter
>    becomes partially invisible. Merged mode is immune.

---

**Tweet 4** — Code to verify it yourself
> ```python
> model_unmerged.eval()  # ← CRITICAL
> logits_a = model_unmerged(**inputs).logits
>
> model_merged = model_unmerged.merge_and_unload()
> model_merged.eval()
> logits_b = model_merged(**inputs).logits
>
> max_diff = (logits_a - logits_b).abs().max().item()
> # healthy: < 1e-3
> # if > 0.01 → check eval(), α/r, device/dtype
> ```
> Run this before certifying any adapter deployment.

---

**Tweet 5** — The practical FDE rule
> Merge for production (single adapter, latency matters, stable weights).
> Keep unmerged for dev, A/B testing, or multi-adapter hot-swap.
>
> If a deployed adapter "has no effect" — check in this order:
> 1. Was model.eval() called?
> 2. Does α/r match training config?
> 3. Run the logit comparison above.
>
> Most null-delta adapters fail check #1.

---

**Tweet 6** — Link to blog
> Full explainer with the forward-pass arithmetic, all 3 divergence
> conditions, and the detection script:
>
> [link to blog post]
>
> Written for @GashawBekele as part of #TRP1 Week 12 gap research.
