# Day 3 — Sources

## Canonical Papers / Primary Sources

1. **Direct Preference Optimization: Your Language Model is Secretly a Reward Model**
   - Authors: Rafailov, R., Sharma, A., Mitchell, E., Ermon, S., Manning, C. D., & Finn, C.
   - Link: https://arxiv.org/abs/2305.18290
   - Why I used it: Primary source for DPO's loss formulation and the implicit reward derivation (Section 4). The gradient structure — which makes DPO a label-faithful transducer with no representation of what chosen/rejected are — is load-bearing for explaining why bias propagates. Also used for the per-token gradient decomposition in Kidane's explainer.

2. **SimPO: Simple Preference Optimization with a Reference-Free Reward**
   - Authors: Meng, Y., Xia, M., & Chen, D.
   - Link: https://arxiv.org/abs/2405.14734
   - Why I used it: Primary source for SimPO's length-normalized reward formulation (Section 3.1, Equation 3). The gradient decomposition showing each token contributes 1/|y| weight is derived directly from differentiating this equation. Load-bearing for all three sub-questions in Kidane's explainer.

---

## Tool or Pattern Used

- **Tool/Pattern:** Criterion rotation audit
- **What I did with it:** Designed a 5-ordering Latin square audit over the tone compliance markers (direct, grounded, honest, professional, non_condescending) to test whether fixed-order labels agree with rotation-averaged majority-vote labels across a 40-pair sample.
- **What I observed:** The audit quantifies the Goodhart tax as the difference between proxy Delta A (fixed-order rubric) and gold Delta A (rotation-averaged rubric). An agreement rate below 0.90 indicates material bias; below 0.70 indicates the labels need to be regenerated before the Delta A figure is defensible.

- **Tool/Pattern:** Mask-and-rescore probe
- **What I did with it:** Masked discriminative tone-marker tokens in held-out preference pairs and rescored the reward gap to test whether the critic's preference survives masking.
- **What I observed:** A survival ratio below 20% indicates a token-localized critic that will generalize to novel body shapes. Above 50% indicates a sequence-diffuse critic that will mis-fire on the v0.2 threaded-trace body shapes.

---

## Follow-on Reading

- Gao et al. (2023). Scaling Laws for Reward Model Overoptimization. https://arxiv.org/abs/2210.10760 — empirical evidence that proxy reward keeps rising while gold reward plateaus; Figure 1 is the canonical illustration of the Goodhart gap in preference learning.
- Wang et al. (2023). Large Language Models are not Fair Evaluators. https://arxiv.org/abs/2305.17926 — position bias mechanism in rubric-style LLM evaluation (from Day 1).
