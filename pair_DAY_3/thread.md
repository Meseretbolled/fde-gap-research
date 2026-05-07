# Day 3 — Tweet Thread

---

**Tweet 1** — Name the question
> I trained a DPO judge on 159 preference pairs and reported +0.1904 Delta A lift.
>
> Then I realized: my tone judge scored criteria in a fixed order. Position bias is real. And DPO is a faithful transducer of whatever the labels encode.
>
> Did I measure genuine improvement — or did I measure the bias twice? 🧵

---

**Tweet 2** — The mechanism
> DPO's loss has no representation of what chosen and rejected *are*.
>
> It only sees the labels.
>
> So if a position-biased judge nudged 20% of your preference labels in a consistent direction, DPO learns that gradient as faithfully as it learns the real outreach-quality signal.
>
> You cannot tell them apart from the loss curve.

---

**Tweet 3** — The Goodhart framing
> This is Goodhart's Law in post-training:
>
> "When a measure becomes a target, it ceases to be a good measure."
>
> Your proxy reward (fixed-order rubric) kept rising.
> Your gold reward (what actual outreach quality is) may have plateaued.
>
> The gap between them is the Goodhart tax. You need to measure it.

---

**Tweet 4** — The audit
> Two cheap experiments, both runnable in an afternoon:
>
> 1. Criterion rotation audit — re-run the judge 5× per pair with different marker orders. If majority-vote label disagrees with original label >10% of the time, the bias is material.
>
> 2. Dual eval — score held-out outputs with fixed-order rubric AND rotation-averaged rubric. The Delta A gap between them is your Goodhart tax, in real units.

---

**Tweet 5** — The SimPO gradient smear (Kidane's question)
> Adjacent finding from my partner @KidaneGebremedhin:
>
> SimPO's length-normalized loss gives every token equal gradient weight (1/|sequence|).
>
> On near-identical pairs, tokens before the tone marker mostly cancel.
> Tokens AFTER it accumulate gradient from context divergence — not quality difference.
>
> Run a mask-and-rescore probe to find out which critic you actually shipped.

---

**Tweet 6** — Link to blog
> Full explainer: the DPO gradient, why label bias propagates, the rotation audit with interpretation thresholds, and the mask-and-rescore probe for SimPO critics.
>
> [Blog link — to be added after publishing]
>
> Written for Kidane Gebremedhin as part of TRP1 Week 12 Day 3 paired gap research.

---

#DPO #SimPO #PreferenceLearning #GoodhartLaw #TRP1 #LLMTraining
