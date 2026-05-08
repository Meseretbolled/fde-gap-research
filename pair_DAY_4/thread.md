# Day 4 — Tweet Thread

**Topic:** When bootstrap is enough — and when you need paired bootstrap for agent benchmarks

---

**Tweet 1** — Name the question
> Your eval shows version B improved success rate from 60% → 73%.
>
> Is that improvement real — or is it just lucky task sampling?
>
> Most agent benchmark write-ups report point estimates and stop there.
> Here's the one method that makes the claim defensible. 🧵

---

**Tweet 2** — Load-bearing mechanism (part 1)
> Simple bootstrap answers: "how much would this metric move if I ran on different tasks?"
>
> It works for characterising ONE version.
> It breaks for COMPARING two versions.
>
> Why? Because tasks have wildly different difficulty.
> Bootstrap version A and B independently and you can accidentally sample hard tasks for one and easy tasks for the other.

---

**Tweet 3** — Load-bearing mechanism (part 2)
> Paired bootstrap fixes this:
> For each resample, draw the SAME task indices for both A and B.
> Compute the DIFFERENCE in metric.
> Build a CI around the difference.
>
> Task difficulty cancels out.
> If the 95% CI excludes zero → the improvement is real.
> If it includes zero → your data doesn't support the claim.

---

**Tweet 4** — Code
> ```python
> RESAMPLES = 10_000
> rng = np.random.default_rng(42)
> diff_samples = []
>
> for _ in range(RESAMPLES):
>     idx = rng.integers(0, N, N)  # same tasks for A and B
>     diff_samples.append(
>         success_b[idx].mean() - success_a[idx].mean()
>     )
>
> lo, hi = np.percentile(diff_samples, [2.5, 97.5])
> # CI excludes 0 → defensible. Includes 0 → not defensible.
> ```
> Run this on your existing score_log.json before claiming a win.

---

**Tweet 5** — The rule + write-up change
> The rule:
> → Single version metric (p95 latency, success rate): simple bootstrap
> → Comparing two versions on the same tasks: paired bootstrap of the difference
> → Different task sets: redesign the eval — pairing is impossible
>
> Before: "Version B improved from 60% to 73%"
> After: "B improved by +13 pp [95% CI: +4 to +22 pp] — CI excludes zero ✓"

---

**Tweet 6** — Link to blog
> Full explainer with the mechanism, runnable code on your score_log.json format, and when 30 tasks is too few to detect real improvements:
>
> [link to blog post]
>
> Written for @Abdulaziz as part of #TRP1 Week 12 gap research.
