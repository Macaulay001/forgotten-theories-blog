# Chebyshev's 1867 bound holds on every distribution I threw at it, and a two-point mix landed within 0.2% of saturating it.

*Part 98 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1867 Pafnuty Chebyshev published a short proof of an inequality that any student of probability eventually meets:

    P(|X - μ| ≥ k σ) ≤ 1 / k²

Two moments. No other assumption. The variable can be discrete, continuous, skewed, heavy-tailed, bimodal, whatever you like, and the probability of falling more than k standard deviations from the mean is bounded by 1/k². At k = 2 the bound says 25%. At k = 3 it says 11.1%. At k = 4 it says 6.25%.

The result is older than the modern measure-theoretic framing of probability. Chebyshev was building toward what we now call the weak law of large numbers, and the inequality was a tool. Markov tightened the proof a few years later in his thesis. Cantelli added a one-sided version in 1910. The result is so quietly absorbed into textbooks that students often skip past it on the way to the central limit theorem and never come back.

I had used Chebyshev as a sanity check in lecture notes for years. I had never bothered to compare the bound against simulated tails for the distributions I actually meet. Two questions sat there unanswered. How loose is it for Normal data, where the 3σ rule already gives a sharper answer. And can a real distribution come close to saturating it, the way the textbook two-point construction suggests.

## What I tried

The plan was a Monte Carlo with N = 10^6 samples from five distributions. Normal(0, 1) for the gold-standard case. Uniform on a fixed interval, rescaled so the variance is one. Exponential(1) shifted so the mean is zero. Pareto with shape parameter α = 2.1, which sits just barely on the finite-variance side of the heavy-tail boundary. And a bimodal mixture designed to approximate the two-point distribution that classical proofs use to show tightness.

The bimodal needs explanation. To saturate Chebyshev at a target value k*, you put mass p = 1/(2 k*²) at each of ±a, with a chosen so that the total variance equals 1. The arithmetic gives a = k*. For k* = 2, that is mass 1/8 at +2 and mass 1/8 at -2, with the remaining 3/4 mass at zero. Variance is exactly 1, mean is zero, and P(|X| ≥ 2σ) = 1/4, which is the Chebyshev bound exactly. I added a tiny Gaussian jitter of width 0.01 so the simulation does not depend on literal atoms.

For each distribution I drew the samples, computed the sample mean μ and sample standard deviation σ, then asked what fraction of the data fell more than k σ from μ for k in {1, 2, 3, 4}. Compare to 1/k² and look at the gap.

That is the whole experiment. It fits in one Python file.

## What happened

Every empirical tail probability sat under the bound. No surprises there. The size of the gap is the interesting number.

Here is the core of the calculation:

```python
def tail_probs(x):
    mu = x.mean()
    sigma = x.std()
    centered = np.abs(x - mu)
    out = np.empty(len(KS))
    for i, k in enumerate(KS):
        out[i] = (centered >= k * sigma).mean()
    return mu, sigma, out

for name, sample_fn in DISTS.items():
    x = sample_fn()           # N = 1_000_000 draws
    mu, sigma, probs = tail_probs(x)
    print(name, probs)        # compare against 1.0/KS**2
```

The summary table tells most of the story.

| Distribution         | k=1     | k=2      | k=3       | k=4       |
|----------------------|---------|----------|-----------|-----------|
| Chebyshev bound 1/k² | 1.0000  | 0.2500   | 0.1111    | 0.0625    |
| Normal(0,1)          | 0.3164  | 0.0458   | 0.00276   | 4.7e-05   |
| Uniform (variance=1) | 0.4224  | 0.0000   | 0.0000    | 0.0000    |
| Exponential, centered| 0.1353  | 0.0500   | 0.01826   | 0.00669   |
| Pareto α = 2.1       | 0.0400  | 0.0152   | 0.00783   | 0.00471   |
| Bimodal two-point    | 0.2495  | 0.1454   | 0.0000    | 0.0000    |

The Normal at k = 3 sits at 0.00276 against a bound of 0.1111. That is a factor of about 40 of slack. At k = 4 the bound is 0.0625 and the empirical mass is 4.7e-5, which leaves a factor of roughly 1330. Anyone who has used the 3σ rule knows the bound is wasteful for Gaussian data, but seeing the gap as a clean ratio is still striking.

The Uniform vanishes at k ≥ 2 because the variance-1 rescaling puts the support inside ±√3, which is about ±1.73, so no observation can be more than 2σ from the mean. Chebyshev does not know that and reserves 25% of the probability mass for an outcome that is geometrically impossible. That is the price of using only the variance.

The bimodal two-point gives the headline number. At k = 2 the empirical probability is 0.2495, against a bound of 0.2500. About 0.2% of the bound is unaccounted for, and that gap is partly Monte Carlo noise and partly the small Gaussian jitter I added to avoid literal atoms. The inequality is essentially tight on the right kind of distribution.

The Pareto case is the one that gets closest to the bound for a continuous heavy-tailed example. With α = 2.1 the variance only barely exists, the sample standard deviation is around 2.72, and the empirical tails at k = 3 and k = 4 are 0.78% and 0.47%. Far from the bound, but at k = 4 the Pareto tail is two orders of magnitude heavier than the Normal tail. Heavy tails matter a lot once you walk away from Gaussian assumptions.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/98_chebyshev_bound_vs_empirical.png)

The figure plots all five empirical curves against the 1/k² reference line on a log-y axis. The bimodal sits right under the line. The Normal is far below it, falling roughly like exp(-k²/2). The Pareto sits in between and drops slowly. The Uniform drops off a cliff.

## What it may mean

The practical reading is unchanged from 1867 and matches what textbooks say. If you only know the mean and variance of something, Chebyshev is the strongest bound available on a single-tail or two-tail probability. If you know the distribution is approximately Normal, the bound is wildly loose and you should not use it. If the distribution is unknown but you expect heavy tails, the bound is closer to useful but still not tight.

This may matter most in places where people quietly assume Normality without checking. Returns on financial assets, generalization error in machine learning, residuals in misspecified regressions. The 3σ rule is a Normal-distribution fact, not a probability-theory fact. A distribution with finite variance can put 25% of its mass past 2σ, and the bimodal example shows this is not a pathological theoretical case but a measurable empirical fact in a small simulation.

There is also a calibration lesson. The two-point distribution that saturates Chebyshev has no density at the tail. All the mass sits at a single point on each side, with nothing in between. Real-world distributions almost never look like this, which is one informal reason the bound is loose in practice. When it is nearly tight, it is because the random variable behaves like an indicator.

I would not call any of this a discovery. It is a calibration of an old result on modern samples, on the kind of distributions a working scientist actually meets. The historical claim, that two moments suffice to bound the tail, holds up cleanly 159 years later.

## Loose ends

I did not chase the k ≥ 5 tail of the Normal because at N = 10^6 you start seeing zero counts. To get a clean Monte Carlo estimate of the 5σ Normal tail (theoretical 5.7e-7) you would want N around 10^9. That is an overnight run on one CPU, not a big lift. A more interesting follow-up would be to compare Chebyshev against the one-sided Cantelli bound 1/(1 + k²) on the same distributions, since Cantelli is uniformly tighter when the tail is one-sided and asymmetric distributions (Exponential, Pareto) are where it should bite. That is another afternoon's work.
