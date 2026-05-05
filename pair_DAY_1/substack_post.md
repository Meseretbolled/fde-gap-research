# Merged vs Unmerged LoRA at Inference: When Are They Identical and When Do They Silently Diverge?

*Published as part of TRP1 Week 12 — Knowledge Gap Research*
*By Meseret Bolled*

---

You trained a LoRA adapter. Loss dropped. Output length fell. Training looked
successful by every metric. But your evaluation scores didn't move at all.

Before blaming the backbone or the dataset — did you check whether your
adapter is actually being applied at inference?

Merged and unmerged LoRA serving are not always identical. Here is the
mechanism, the three conditions that break it, and the practical rule
for every adapter deployment.

---

## What Are Merged and Unmerged LoRA?

A LoRA adapter adds two low-rank matrices — **B** and **A** — to a frozen
base weight matrix **W**. During training, the effective weight is:

```
W_eff = W + (α/r) × B × A
```

At inference time, there are two ways to apply this.

**Unmerged (dynamic application)**
The adapter branch runs as a separate computation on every forward pass:

```
h = W·x + (α/r) · B · A · x
```

The base weights are never touched. The LoRA branch is added to the base
output at each layer, every time you run the model.

**Merged (permanent fusion)**
Before serving starts, the adapter is fused once into the base weights:

```
W' = W + (α/r) × B × A      ← done once, offline
h  = W' · x                  ← standard forward pass, no extra branch
```

From the model's perspective, the merged model looks identical to a
fully fine-tuned base model. The LoRA matrices no longer exist as
separate objects.

**Are they mathematically identical?**
Algebraically: yes. `W'·x = W·x + (α/r)·B·A·x` by the distributive law.
In practice: almost always — but three real conditions break this.

---

## Three Conditions That Cause Silent Divergence

### 1. Floating-point rounding (fp16/bf16)

In unmerged mode, two separate matrix multiplies accumulate rounding errors
independently before being summed. In merged mode, the addition happens once
at high precision and is stored. The numerical gap is typically less than
1e-3 in logit space — below the threshold that changes top-k token selection.
For a small model with low rank this is negligible. It becomes material only
at larger ranks or when accumulated across many layers.

### 2. Scaling factor misapplication

The scale `α/r` must be applied identically in both paths. A common
implementation error is to apply it during the merge step but not during
dynamic unmerged inference — or vice versa. This produces a systematic
output shift. Not a floating-point rounding error. A wrong answer. Always
verify your PEFT library version applies the scaling consistently in both
code paths.

### 3. Dropout left on at inference ← the most dangerous one

LoRA uses dropout on the A matrix during training. At inference, dropout
must be disabled by calling `model.eval()`. If you forget this in unmerged
mode, the A branch randomly zeroes activations on every forward pass —
making the adapter non-deterministic and partially invisible. Merged mode
is immune because the adapter no longer exists as a separate module.

This is the most likely silent failure mode for an adapter that
"appears to have no effect" after deployment.

---

## How to Verify Your Adapter Is Actually Active

Run both modes on the same prompt and compare the logits directly:

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import PeftModel
import torch

BASE_ID    = "your-base-model"
ADAPTER_ID = "your-adapter"
PROMPT     = "your test prompt"

tokenizer = AutoTokenizer.from_pretrained(BASE_ID)
inputs    = tokenizer(PROMPT, return_tensors="pt")

# Unmerged
base_model     = AutoModelForCausalLM.from_pretrained(BASE_ID, torch_dtype=torch.float16)
model_unmerged = PeftModel.from_pretrained(base_model, ADAPTER_ID)
model_unmerged.eval()  # ← CRITICAL

with torch.no_grad():
    logits_unmerged = model_unmerged(**inputs).logits

# Merged
model_merged = model_unmerged.merge_and_unload()
model_merged.eval()

with torch.no_grad():
    logits_merged = model_merged(**inputs).logits

# Compare
max_diff  = (logits_merged - logits_unmerged).abs().max().item()
top1_same = (logits_merged.argmax(-1) == logits_unmerged.argmax(-1)).all().item()

print(f"Max logit difference : {max_diff:.6f}")  # healthy: < 1e-3
print(f"Top-1 token matches  : {top1_same}")      # healthy: True
```

If `top1_same` is `False` or `max_diff` is greater than 0.01, check in
this order:
1. Was `model.eval()` called before both forward passes?
2. Is the same `lora_alpha / r` scaling used in both paths?
3. Are both models on the same device and dtype?

---

## The Practical FDE Rule

**Merge when:**
- Serving a single adapter at scale and latency matters
- The adapter is stable — no more A/B testing or hot-swapping
- Memory is constrained — merged model is the same size as the base model

**Keep unmerged when:**
- Hot-swapping adapters across multiple clients on one GPU
- Still A/B testing the adapter against a baseline
- Running the logit comparison above to verify the adapter is active

**If a deployed adapter appears to have no effect — check in this order:**
1. Was `model.eval()` called?
2. Does `lora_alpha / r` match the training config?
3. Run the logit comparison — if `max_diff ≈ 0` everywhere, the adapter
   weights may be zero-initialised and training did not save correctly
4. Are the adapter and base model on the same device and dtype?

---

## Sources

- Hu et al. (2021). *LoRA: Low-Rank Adaptation of Large Language Models.*
  https://arxiv.org/abs/2106.09685

- Hugging Face PEFT Documentation.
  https://huggingface.co/docs/peft/conceptual_guides/lora

---

*Written for Gashaw Bekele as part of TRP1 Week 12 paired gap research.
Gashaw trained a LoRA adapter (r=16, α=16, Qwen2.5-0.5B) whose rubric
scores showed Delta A = 0.00. This explainer covers the serving-mode
diagnostic layer — the second thing to check after backbone capacity.*
