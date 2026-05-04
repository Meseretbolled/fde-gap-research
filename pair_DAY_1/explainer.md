# explainer.md

**Explainer:** Meseret Bolled
**Asker:** Gashaw Bekele
**Topic area:** Inference-time mechanics — LoRA adapter serving
**Date:** 2026-05-04

---

## 1. The Forward-Pass Arithmetic: Merged vs Unmerged

A LoRA adapter adds two low-rank matrices — **B** (d × r) and **A** (r × k) —
to a frozen base weight matrix **W** (d × k). During training, the effective
weight is:

```
W_eff = W + (α/r) × B × A
```

where `α` is the LoRA scaling factor and `r` is the rank. At inference time
there are two ways to apply this:

**Unmerged (dynamic application)**
The adapter branch is computed separately on every forward pass:

```
h = W·x + (α/r) · B · A · x
```

The base weights **W** are never modified. The LoRA branch runs as a parallel
path and its output is added to the base output at each layer.

**Merged (permanent fusion)**
Before generation starts, the adapter is fused once into the base weights:

```
W' = W + (α/r) × B × A      ← done once, offline
h  = W' · x                  ← standard forward pass, no extra branch
```

From the model's perspective, the merged model looks identical to a
fine-tuned base model — the LoRA matrices no longer exist as separate objects.

**Are they mathematically identical?**
Algebraically: yes. `W'·x = W·x + (α/r)·B·A·x` by the distributive law.
In practice: almost always yes, but with three known exception conditions
covered in the next section.

---

## 2. Conditions Where the Two Modes Can Diverge

### 2a. Floating-point rounding (fp16/bf16)

fp16 has ~3 decimal digits of precision; bf16 has ~2.5. When merging,
the addition `W + (α/r)·BA` is computed once at high precision and stored.
At runtime, `W'·x` involves one matrix multiply.

In unmerged mode, `W·x` and `(α/r)·B·A·x` are computed separately and then
added. Two matrix multiplies accumulate rounding errors independently before
summation. The numerical gap is typically < 1e-3 in logit space — below the
threshold that changes top-k token selection — but it is non-zero. For a
0.5B model with r=16 this gap is negligible; it becomes material only on
larger ranks or accumulated across many layers with accumulated residuals.

### 2b. Scaling factor misapplication (most common real-world bug)

The scale `α/r` must be applied identically in both paths. A common
implementation error is to apply it during the merge step but not during
dynamic unmerged inference (or vice versa). This produces a systematic
output shift — not a floating-point error but a wrong answer. Always verify
the PEFT library version applies `lora_alpha / r` consistently in both code
paths.

### 2c. Dropout left on at inference

LoRA uses dropout on the A matrix during training. At inference, dropout
must be disabled (`model.eval()`). If `model.eval()` is not called before
generation in unmerged mode, the A branch randomly zeroes activations,
making the adapter non-deterministic and partially invisible. Merged mode
is immune because the adapter no longer exists as a separate dropout-bearing
module. This is the most likely silent failure mode for an adapter that
"appears to have no effect" after deployment.

---

## 3. Code Example: Logit Comparison

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import PeftModel
import torch

BASE_ID    = "unsloth/Qwen2.5-0.5B-Instruct"
ADAPTER_ID = "gashawbekele/tenacious-bench-lora-path-a"
PROMPT     = "Draft a one-sentence outreach for a prospect who posted 8 ML roles."

tokenizer = AutoTokenizer.from_pretrained(BASE_ID)
inputs    = tokenizer(PROMPT, return_tensors="pt")

# ── Unmerged inference ──────────────────────────────────────────────────────
base_model      = AutoModelForCausalLM.from_pretrained(BASE_ID, torch_dtype=torch.float16)
model_unmerged  = PeftModel.from_pretrained(base_model, ADAPTER_ID)
model_unmerged.eval()                          # ← CRITICAL: disables dropout

with torch.no_grad():
    logits_unmerged = model_unmerged(**inputs).logits

# ── Merged inference ────────────────────────────────────────────────────────
model_merged = model_unmerged.merge_and_unload()   # fuses B×A into W, removes PEFT wrappers
model_merged.eval()

with torch.no_grad():
    logits_merged = model_merged(**inputs).logits

# ── Compare ─────────────────────────────────────────────────────────────────
max_diff  = (logits_merged - logits_unmerged).abs().max().item()
top1_same = (logits_merged.argmax(-1) == logits_unmerged.argmax(-1)).all().item()

print(f"Max logit difference : {max_diff:.6f}")   # healthy: < 1e-3
print(f"Top-1 token matches  : {top1_same}")       # healthy: True
```

If `top1_same` is `False` or `max_diff` > 0.01, check: (1) was `eval()` called
before both forward passes, (2) is the same `lora_alpha` / `r` being used,
(3) are both tensors on the same device and dtype.

---

## 4. Practical FDE Rule

**Merge when:**
- Serving a single adapter at scale and latency matters
- The adapter is stable (no more A/B testing or hot-swapping)
- Memory is constrained — merged model = base model size, no extra B/A matrices

**Keep unmerged when:**
- Hot-swapping adapters across multiple clients on one GPU
- Still A/B testing the adapter against baseline or prompt-engineering
- Running the logit comparison above to verify the adapter is active

**If an adapter appears to have no effect after deployment — check in this order:**
1. Was `model.eval()` called? (dropout check)
2. Print `lora_alpha` and `r` — confirm `α/r` scaling matches training config
3. Run the logit comparison above — if `max_diff` ≈ 0 at all positions the
   adapter weights are likely zero-initialised and training did not save
4. Check the adapter was loaded to the same device and dtype as the base model

**On Gashaw's specific null delta (Delta A = 0.00):**
The serving mode is not the primary cause. The `methodology_rationale.md`
diagnosis is correct: a 0.5B backbone attention-copies "bench" from the
input `bench_summary` field because the token appears in the prompt and
the model's capacity is insufficient to suppress it via weight updates
alone. The adapter learned concision (−18% length, loss 3.08 → 0.42) but
could not override a strong attention signal from a token present in the
context window. The serving-mode check is still worth running as a
second-order verification, but it will not move the rubric score until the
backbone capacity bottleneck is resolved (e.g., Qwen2.5-1.5B in v0.2).

---

## Summary

| | Merged | Unmerged |
|---|---|---|
| **Arithmetic** | W' = W + (α/r)BA, then h = W'x | h = Wx + (α/r)BAx |
| **Logit difference** | — | < 1e-3 (fp16 rounding only) |
| **Dropout risk** | None — no adapter module | Yes — must call `.eval()` |
| **Latency** | Same as base model | Slightly higher (extra matmul) |
| **Memory** | Same as base model | Base + B + A matrices |
| **Best for** | Production single-adapter serving | Dev, A/B testing, multi-adapter |
| **Silent failure mode** | Scaling bug at merge time | `eval()` not called |
