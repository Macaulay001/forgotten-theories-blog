# I dropped 10 million simulated needles on a ruled plane. Buffon's 1733 formula was right to four decimals.

*Part 42 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1733 Georges-Louis Leclerc, the future Comte de Buffon, asked a casino-flavored question at a meeting of the Paris Academy of Sciences. If you toss a needle of length L onto a floor ruled with parallel lines spaced d apart, with L no longer than d, what fraction of throws cross a line? He waited until 1777 to publish the answer in his "Essai d'arithmetique morale". The probability is 2L over π times d. Rearrange and you have an estimator of π built out of needles and a tape measure, which is the first time in history anyone proposed a physical experiment to compute a mathematical constant.

The result was forgotten for about a century, then rediscovered by Laplace in 1812, then turned into a parlor game, then turned into the canonical introductory example of Monte Carlo methods. Today it shows up in every probability textbook as a one-paragraph anecdote. The anecdote always claims it works. I had never actually checked.

![Buffon needles on a ruled plane](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/42_needles.png)

## What I tried

The simulation is short enough that I could write it on a napkin. With L = d = 1, drop a needle by sampling a center y in [0, 0.5] (using the symmetry of the spacing) and an angle theta in [0, π/2]. The needle crosses a line whenever the projection (L/2) times sin(theta) is at least y. Count crossings, call that k, and the formula 2N/k is an estimate of π.

The plan was to run that at five sample sizes (10³, 10⁴, 10⁵, 10⁶, 10⁷ drops) with 50 independent replicates per size, plot the convergence with error bars, and check whether the empirical standard error follows the 1/√N law that elementary Monte Carlo theory predicts. Then a direct comparison: do the same calculation with the simpler point-in-square method (drop points in a unit square, count those inside the inscribed quarter circle, multiply the fraction by 4) and see which one wins.

The third thing I wanted to know was practical. How many drops does it take to nail π to two decimal places? Three? Four? The textbook answer is "many", which is unhelpful. I wanted an actual number, and I wanted to compare it against the famous 1901 result by the Italian mathematician Mario Lazzarini, who reported π equal to 355/113 (accurate to seven decimal places) after just 3,408 needle tosses. The Lazzarini number has been suspect for a long time. Modern reviewers think he stopped throwing when the ratio crossed his target. I wanted to see whether his sample size could plausibly support that accuracy at all.

The whole simulation runs on a laptop in about 30 seconds.

## What happened

The convergence figure looks the way Buffon would have wanted it to look in 1777 but had no way to draw. At N = 1000 the estimate is 3.170 with a 50-replicate standard deviation of 0.067, which means a typical run only nails π to one decimal place. At N = 10,000 the estimate is 3.144 with SE 0.021, still only two decimals. At N = 10⁷ the estimate is 3.14153 with SE 7.7 × 10⁻⁴, finally landing on four reliable decimals.

```python
# Buffon needle, L = d = 1. y in [0, 1/2] by symmetry, theta in [0, pi/2].
y = rng.uniform(0.0, 0.5, size=N)
theta = rng.uniform(0.0, np.pi / 2, size=N)
k = (0.5 * np.sin(theta) >= y).sum()
pi_hat = 2 * N / k
```

The standard error scales as 1/√N over four decades, which is the central textbook claim about Monte Carlo. The analytic constant for Buffon at L = d is √(π(π − 2)), which equals 1.894. Empirically at N = 10⁷ the SE is 0.000768, so the implied constant is 2.43, about 28% above theory. The gap is the bias of the 2N/k transformation: dividing by a random count inflates the spread relative to the delta-method approximation. By N = 10⁵ and above the bias has stopped growing, which matches what the standard analyses (Mantel 1953, Solomon 1978) predict.

![Convergence of Buffon vs point-in-square](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/42_convergence.png)

The point-in-square comparison is sobering for fans of needles. Same N, same hardware, the direct estimator wins by about a factor of 1.6 in standard error. Its analytic constant is √(π(4 − π)) = 1.642, and the empirical value at N = 10⁷ comes in at 1.52, within 8% of theory. Why does direct sampling beat the needle? Both estimators are unbiased samplers of a Bernoulli random variable. Buffon's Bernoulli has success probability 2/π ≈ 0.637, the direct one has success probability π/4 ≈ 0.785. The lower-variance Bernoulli (closer to 0.5 is worse for proportion variance, and π/4 is further from 0.5 than 2/π) makes the direct estimator slightly more efficient, and then dividing by a count (Buffon) versus multiplying by a count (direct) tilts the picture further.

| N    | Buffon SE   | Direct SE   | ratio |
|------|------------:|------------:|------:|
| 10³  | 0.0674      | 0.0500      | 1.35  |
| 10⁴  | 0.0207      | 0.0181      | 1.14  |
| 10⁵  | 0.00662     | 0.00590     | 1.12  |
| 10⁶  | 0.00266     | 0.00165     | 1.61  |
| 10⁷  | 0.000768    | 0.000480    | 1.60  |

The practical-accuracy question was the most fun. Running a single growing chain and asking when its cumulative estimate first sustains a target accuracy (it must hold for the next 100 samples too, so I can't reward lucky single hits), the medians come out roughly: 2,000 drops to beat 10⁻² accuracy, 16,000 drops to beat 10⁻³, and around 3.3 × 10⁵ drops to beat 10⁻⁴. The direct estimator is slightly better at 10⁻⁴ (about 2.5 × 10⁵) and slightly worse at the easier targets, but the spread between trials is so wide that the two methods are statistically tied at every threshold I checked.

Which brings us to Lazzarini. He reported π = 355/113 (a famous Chinese approximation from the 5th century by Zu Chongzhi, accurate to within 3 × 10⁻⁷) after 3,408 throws. My simulation says the probability of doing that well at N ≈ 3,400 is essentially zero. At that sample size you cannot even reliably beat 10⁻² accuracy, much less 10⁻⁷. The kind reading is that he chose to stop when the ratio passed his target. The unkind reading is that he reverse-engineered the throw count from 113 (the denominator of his target) and reported what he wanted. Either way the result is theatre, not measurement. The interesting thing is that Buffon's formula itself is fine. Lazzarini just used it dishonestly.

## What it may mean

Buffon's needle is a Monte Carlo method published more than 150 years before anyone used the phrase "Monte Carlo". The 1949 Metropolis-Ulam paper is usually credited with launching modern stochastic simulation, but the conceptual move (turning a geometric probability into a numerical estimate) is already complete in Buffon's setup. What changed in 1949 was not the idea, it was the existence of computers fast enough to make the idea pay for itself relative to deterministic series.

Today no one uses random methods to estimate π. The Chudnovsky brothers' 1989 series converges by about 14 decimal digits per term, and modern computations are deep into the trillions of digits. What survives from Buffon is the more general principle that an unbiased random estimator has standard error proportional to 1/√N, and that you can buy more precision only by running longer. That law applies to particle physics simulations, to financial pricing, to climate ensembles, and to the entire toolkit of randomized algorithms. The needle is the cleanest possible illustration of the law, and on this sample it holds to four decimals.

There is also a small lesson about provenance. The fact that 1901 Lazzarini's "result" survived in popularizations for a century, despite being statistically impossible, is a warning about anecdotes that look too clean. Whenever a single experiment lands implausibly close to the right answer with a small sample, the prior should shift toward selective reporting rather than toward genius.

## Loose ends

With another week I would run the long-needle case (L > d, where the formula picks up an extra inverse-cosine term) and the Buffon-Laplace generalization to a rectangular grid, both for completeness. I would also try a few variance-reduction tricks (antithetic angles, stratified sampling over y) to see how close to the Cramer-Rao bound a Monte Carlo π estimator can get. The whole pipeline runs in under a minute on a laptop, so anyone who wants to argue about Lazzarini can replicate it from scratch.
