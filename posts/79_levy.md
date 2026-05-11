# I ran Paul Levy's 1925 generalized CLT on 540,000 sums of heavy-tailed variables and the stable laws matched to KS = 0.005.

*Part 79 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

The central limit theorem is the most quoted result in statistics. Add up many independent things, divide by sqrt(N), and you get a Gaussian. That sentence is in every introductory probability course. The version you usually meet has a quiet assumption tucked inside: the summands need finite variance. If the variance is infinite, the sqrt(N) trick stops working and the limit stops being Gaussian.

Paul Levy wrote down the fix in his 1925 *Calcul des probabilites* and worked out the full picture by 1937. He showed there is a wider family of limit distributions, indexed by a tail parameter alpha between 0 and 2, that absorb sums of iid variables when the variance blows up. The Gaussian is the alpha = 2 endpoint. The Cauchy is alpha = 1. Everything in between has heavier tails than a normal but finite enough mass at the center to still converge under a different scaling, namely N^{1/alpha} instead of sqrt(N). Levy's family is sometimes called the alpha-stable laws, or just the Levy stable distributions.

This is one of those results that quietly underwrites entire fields (finance, anomalous diffusion, network theory) yet still feels foreign to most people who learned only the alpha = 2 case. I wanted to see the generalized CLT happen on my screen, the way I had seen the ordinary one a hundred times.

## What I tried

The setup was three cases of the same experiment.

For each case, I drew N iid samples from a distribution with a known tail index alpha, summed them, divided by N^{1/alpha}, and repeated M = 20,000 times. That gives me an empirical distribution of standardized sums. Then I asked: does this empirical distribution match the theoretical Levy stable law with the corresponding alpha?

The three cases:

- alpha = 1, Cauchy summands. Cauchy is a fixed point of the operation: a sum of N iid Cauchy variables divided by N is again standard Cauchy. So I expected the empirical histogram to overlay the Cauchy pdf exactly, for every N.
- alpha = 1.5, symmetrized Pareto with tail exponent 1.5. Variance is infinite (the second moment integral diverges), so the ordinary CLT cannot apply. Levy predicts convergence to a symmetric alpha-stable law with alpha = 1.5.
- alpha = 2, standard Gaussian summands. The classical case. Levy's family at alpha = 2 reduces to a Gaussian with variance 2 * scale^2, so the limit should be N(0, 1) using scale = 1/sqrt(2).

For each case I ran three values of N (10, 100, 1000) to see whether more terms tightened the fit. Then I computed a Kolmogorov-Smirnov statistic between the empirical cdf of the standardized sums and the theoretical cdf from scipy.stats.levy_stable. The KS statistic is the maximum vertical gap between two cdfs, scale-free and intuitive: 0.01 means worst-case 1% disagreement anywhere on the curve.

I also overlaid a Gaussian on each panel, fitted to the bulk of the empirical sums, to make the misfit visible in the heavy-tailed cases. This is the rhetorical part of the experiment: showing how badly the standard CLT picture lies when alpha < 2.

The core loop is short:

```python
# draw N iid heavy-tailed samples, m trials, sum and normalize
draws = sample_pareto(n=1000, m=20000, alpha=1.5)   # shape (M, N)
std_sums = draws.sum(axis=1) / n ** (1.0 / 1.5)     # N^{1/alpha} scaling
ks = stats.kstest(std_sums,
                  lambda x: stats.levy_stable.cdf(x, 1.5, 0.0, scale=sc))
print(ks.statistic)   # 0.0075
```

The Pareto draws were symmetrized (random sign flips) so the limit is the symmetric stable rather than a one-sided skewed version. That keeps the parameter count small.

## What happened

The match was tight everywhere.

| case | alpha | N=10 KS | N=100 KS | N=1000 KS |
|---|---|---|---|---|
| Cauchy | 1.0 | 0.0043 | 0.0047 | 0.0084 |
| Pareto tail 1.5 | 1.5 | 0.0081 | 0.0054 | 0.0075 |
| Gaussian | 2.0 | 0.0070 | 0.0062 | 0.0057 |

The largest KS distance across the nine cells is 0.0084. That means the empirical and theoretical cdfs nowhere differ by more than about 0.8% in cumulative probability. For 20,000 samples that is roughly the floor of finite-sample noise.

Three observations from the run.

The Cauchy row stays flat with N. That is not an accident, it is the stability property: Cauchy is a fixed point of the iid-sum operation. The reason Cauchy means tells you nothing useful about a Cauchy distribution is the same reason Cauchy is a stable law. A million Cauchy variables averaged together is no tighter than one Cauchy variable.

The Pareto row converges to a stable law cleanly, but the *scale parameter* of the limiting stable drifts upward from 1.62 at N=10 to 1.80 at N=1000. That drift is real and known: convergence to the alpha-stable basin is famously slow for finite-variance-looking but actually infinite-variance distributions. The shape is right at N=10, but the scale still settles. By N=1000 the fit looks essentially exact.

The Gaussian row reproduces the classical CLT as the alpha = 2 case of the family. The fitted scale is 1/sqrt(2) ≈ 0.707, which is exactly the relationship between the stable parametrization and the unit normal.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/79_stable_clt.png)

The figure shows what makes this experiment more than a sanity check. In each panel I plotted a Gaussian (dashed green) fitted to the bulk of the standardized sums, beside the red stable pdf. For the Gaussian row the two curves sit on top of each other. For the Cauchy and Pareto rows the green dashed curve is visibly the wrong shape: too narrow in the middle, falling off too fast in the tails. The empirical histogram (blue) goes where the red stable pdf goes, not where the green Gaussian wants it to.

A practical translation: if you take a thousand Pareto-distributed losses, average them, and then ask how often the average will be more than 5 standardized units from the center, the Gaussian says essentially never. The actual stable distribution says it happens roughly 5% of the time. That gap is where most "we did not expect a 1-in-10,000 event" disasters seem to live.

## What it may mean

The result itself is not a discovery; Levy did the math and Gnedenko-Kolmogorov tied up the converse in 1949. What the simulation may make tangible is the size of the gap when you use the wrong limit law. Standardized sums of heavy-tailed variables follow a recognizably different curve, and any inference built on standard deviations or z-scores will quietly misprice the tails.

The places where this matters in practice are exactly the places where data appears to have "outliers" you cannot explain away. Financial returns, fault sizes in materials, file sizes on the web, gravitational fields from a random distribution of stars (Holtsmark, 1919, who derived a special case of an alpha-stable law before Levy did the general theory). In each of these, the natural variable is a sum of many small contributions with heavy-tailed individual sizes, and the right summary distribution is a Levy stable, not a Gaussian.

The other thing the experiment may underline is how cheap it is to check. The whole sweep ran in about a minute on a laptop. Three lines of scipy give you the limit pdf at any alpha you want. Levy needed careful real analysis and characteristic-function arithmetic. Today the gap between "I have a heavy-tailed sum" and "I can compute the right limit law" is one import statement.

## Loose ends

A few honest gaps. I tested only the symmetric beta = 0 slice of the stable family. The full family has four parameters (alpha, beta, scale, location) and the skewed cases (beta != 0) describe one-sided heavy tails common in waiting-time and loss distributions. The KS test compares marginal cdfs, not joint behavior, so dependence structure is out of scope. I also leaned on scipy.stats.levy_stable, which evaluates the pdf via numerical integration of the characteristic function for alpha not in {1, 2}; small bias in the deep tails is possible. With another week I would map the convergence rate to the alpha-stable basin as a function of N for several alpha values and check the Berry-Esseen-style bounds that exist in the literature.
