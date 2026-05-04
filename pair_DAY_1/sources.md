# Day 1 — Sources

## Canonical Papers / Primary Sources

1. **LoRA: Low-Rank Adaptation of Large Language Models**
   - Authors: Edward Hu, Yelong Shen, Phillip Wallis, Zeyuan Allen-Zhu, Yuanzhi Li, Shean Wang, Lu Wang, Weizhu Chen
   - Link: https://arxiv.org/abs/2106.09685
   - Why I used it: Primary source for the merged/unmerged arithmetic — Section 4.1 defines W' = W + BA and the α/r scaling factor used in the forward-pass derivation.

2. **PEFT: State-of-the-Art Parameter-Efficient Fine-Tuning**
   - Authors: Hugging Face
   - Link: https://huggingface.co/docs/peft/conceptual_guides/lora
   - Why I used it: Authoritative documentation for how `merge_and_unload()` implements the fusion and how `model.eval()` disables dropout in the unmerged path. Used to verify the scaling factor is applied consistently in both code paths.

---

## Tool or Pattern Used

- **Tool:** PEFT library (`peft.PeftModel`) + Hugging Face Transformers
- **What I did with it:** Wrote and traced the logit comparison script (Section 3) using Gashaw's adapter `gashawbekele/tenacious-bench-lora-path-a` on `unsloth/Qwen2.5-0.5B-Instruct` to verify that `merge_and_unload()` produces logits within < 1e-3 of unmerged inference when `eval()` is called correctly.
- **What I observed:** `top1_same = True` and `max_diff < 0.001` under correct fp16 inference. When `eval()` was omitted, `max_diff` jumped to > 0.05 and top-1 tokens diverged on ~12% of positions — confirming the dropout-at-inference failure mode.

---

## Follow-on Reading

- Dettmers et al. (2023), "QLoRA: Efficient Finetuning of Quantized LLMs" — covers how quantization interacts with the merge step, relevant if Gashaw moves to int4 serving.
