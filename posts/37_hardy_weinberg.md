# I simulated one generation of random mating from a skewed start. Hardy's 1908 identity reappeared exactly.

*Part 37 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1908 a British mathematician named G.H. Hardy wrote a short letter to *Science* to clean up what he thought was an obvious error in a biology paper. The biology paper had argued that a dominant allele would, generation by generation, drive a recessive allele out of a population. Hardy disagreed. He showed in about a page of algebra that if mating is random and there is no selection, drift, migration, or mutation, then a population with allele frequencies p and q = 1 − p will have genotype frequencies p², 2pq, q² after exactly one generation, and those frequencies will stay there forever.

The same identity had been derived a few months earlier by a German physician, Wilhelm Weinberg, in a longer paper that nobody outside the German medical literature read for two decades. Hardy's two-page note carried the day in English, and the joint name stuck.

It is the foundational identity of population genetics. Every GWAS quality-control pipeline today still tests for departures from HW in controls as a check on genotyping error. I had taught the formula for years and never actually run a generation of random mating from a non-HW starting state to watch it snap into place.

## What I tried

I wanted three things in one pass.

First, a clean demonstration that one generation of random mating from a non-HW starting state lands exactly on (p², 2pq, q²). The trick is to pick starting genotype frequencies that have the desired allele frequency but are *not* in HW, so the convergence is visible. I chose (AA, Aa, aa) = (0.05, 0.50, 0.45), which has p = 0.30 but an HW prediction of AA = 0.09. If the theory is right, after one round of random mating the AA frequency should jump from 0.05 to 0.09 and stay there.

Second, a Wright-Fisher drift simulation at three population sizes (N = 100, 1000, 10000) starting from p = 0.5, with 50 replicates each and 2000 generations. The diffusion approximation says one-generation variance in allele frequency should equal p(1 − p)/(2N), and the eventual fixation probability of the A allele should equal its starting frequency. Both are testable.

Third, a selection simulation. Add a selection coefficient s against the recessive aa homozygote, so its relative fitness is 1 − s. Run with s = 0.01, 0.05, 0.10 for 1000 generations at N = 10⁴, alongside the deterministic recursion. The deterministic curve and the simulated mean should agree within stochastic noise. Post-selection genotype frequencies should also show a small but visible departure from HW: the heterozygote excess that population geneticists use as a fingerprint of purifying selection.

The whole simulator is a couple of hundred lines of NumPy. The drift step is a single binomial draw per generation. The selection step is an explicit one-line viability recursion. No tricks.

```python
# Wright-Fisher drift: one binomial draw per generation
def wright_fisher(N, p0, gens, reps):
    out = np.zeros((reps, gens + 1))
    out[:, 0] = p0
    for r in range(reps):
        p = p0
        for t in range(1, gens + 1):
            k = rng.binomial(2 * N, p)
            p = k / (2 * N)
            out[r, t] = p
    return out
```

That, plus a deterministic recursion for the selection arm, is the entire engine.

![Hardy-Weinberg genotype curves with the simulated single-generation point overlaid at p = 0.30.](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/37_hw_genotype_curves.png)

## What happened

The infinite-population check came out exact, as it had to. Starting from (0.05, 0.50, 0.45), one generation of random mating gives (0.09, 0.42, 0.49). The maximum absolute deviation from (p², 2pq, q²) at p = 0.30 is 0 to printed precision. To rule out a bug, I redid the check stochastically with N = 10⁶ individuals where each individual is two independent gamete draws from a pool with allele frequency p. The realized counts came out (0.0898, 0.4202, 0.4900), within 2 × 10⁻⁴ per cell, which is the right size for √(1/N) sampling noise.

The drift variance check was the most satisfying. At each N I ran 20,000 one-step simulations and measured the variance of the resulting allele frequency. The ratio of empirical to theoretical variance came out 1.007, 1.006, 1.001 for N = 100, 1000, 10000. That is within Monte Carlo noise across three orders of magnitude in N.

| N      | empirical one-step var | p(1−p)/(2N) | ratio |
|-------:|-----------------------:|------------:|------:|
|    100 |             1.26 × 10⁻³ |  1.25 × 10⁻³ | 1.007 |
|   1000 |             1.26 × 10⁻⁴ |  1.25 × 10⁻⁴ | 1.006 |
|  10000 |             1.25 × 10⁻⁵ |  1.25 × 10⁻⁵ | 1.001 |

The long-run drift trajectories also behaved. At N = 100, every replicate had fixed or lost the allele by generation 2000, which is the expected behavior when t/N ≫ 1. At N = 1000 about half the replicates had fixed and the other half still drifted. At N = 10000 not a single replicate had fixed by 2000 generations, and the mean of the surviving polymorphic frequencies was 0.498, indistinguishable from the starting 0.5.

![Drift trajectories at N = 100, 1000, 10000 starting from p = 0.5, 50 replicates each. Small populations fix fast; large ones barely move.](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/37_drift_trajectories.png)

The selection arm matched the deterministic recursion within the size of the stochastic spread. At s = 0.01 the deterministic prediction is p = 0.898 after 1000 generations; the stochastic mean over 20 replicates at N = 10⁴ came in at 0.894. At s = 0.05 the numbers were 0.979 and 0.975. At s = 0.10 they were 0.990 and 0.993. The stochastic value sits slightly above the deterministic at s = 0.10 because the recessive aa class is being driven so close to zero that the binomial step is no longer a small perturbation, and the rare-allele dynamics get noisier.

The departure-from-HW signature appeared exactly where it should. After one round of viability selection with s = 0.10 starting from a HW population at p = 0.5, the post-selection genotype frequencies are (0.256, 0.513, 0.231). The HW reconstruction from the new allele frequency p = 0.513 would be (0.263, 0.500, 0.237). The heterozygote class is in excess by 1.3 percentage points, which is the standard textbook signature of selection against a recessive homozygote. One more round of random mating wipes the excess out and the population returns to HW at the new p. Selection breaks HW only transiently; random mating re-establishes it every generation.

## What it may mean

Hardy's 1908 letter solves a problem that almost no one today would even notice as a problem. The biology paper Hardy was responding to had reasoned that since dominant alleles "win" in heterozygotes, they should accumulate over generations. That intuition is wrong in a specific way: it confuses *expression* with *transmission*. Dominance affects what a heterozygote looks like, not which alleles it passes on. The two alleles segregate symmetrically into gametes, and so allele frequencies do not change just because one allele is dominant.

The 1908 identity is now the null distribution against which everything else gets measured. Modern GWAS pipelines reject SNPs whose control-population genotype frequencies depart from HW at p < 10⁻⁶, because that signal usually means the genotyping cluster plot is broken (Anderson et al. 2010, *Nat Protoc*). Forensic DNA match probabilities multiply per-locus HW frequencies. The Wright-Fisher diffusion that this audit recovers at the 1% level is the basis of every coalescent simulator used in human evolutionary genetics.

What is striking, on reflection, is how little machinery the 1908 result needs. It is a one-line algebraic identity that survived a century of model elaboration intact. The audit reproduces it at machine precision in the deterministic case and at sampling-noise precision under finite N. Selection and drift each pull the system away in the directions predicted by Wright and Fisher in the 1930s. None of this is news. It still feels good to watch it work.

## Loose ends

With another week I would extend the selection arm to overdominant fitness (heterozygote advantage), which produces a stable interior equilibrium and is the cleanest illustration of why selection plus diploidy can maintain polymorphism for millions of years. The sickle-cell allele in malarial regions is the textbook real-world case. I would also run a small two-locus simulation with linkage to watch HW being maintained at each locus while linkage disequilibrium decays at rate r per generation. The whole replication runs in under a minute on a laptop.
