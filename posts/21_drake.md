# I redid Drake's 1961 chalkboard math with a million Monte Carlo samples. His median guess survived.

*Part 21 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In November 1961, at a small meeting in Green Bank, West Virginia, Frank Drake wrote seven symbols on a chalkboard and multiplied them together. The product was supposed to estimate N, the number of communicating civilizations in the Milky Way. Drake's own answer, depending on which term he stressed, came out around 10. He later admitted the equation was meant as a meeting agenda, not a measurement. Each factor was a topic of discussion, and the multiplication was a way of saying "these are the things we would need to know."

Sixty-five years later, two of the seven terms have actual numbers attached. Kepler told us that almost every Sun-like star has planets, and that roughly 10 to 40 percent of them carry a rocky world in the habitable zone. The other five terms are guesses with error bars that span orders of magnitude. I wanted to see what happens if you write Drake's equation honestly, treat every term as a probability distribution instead of a point estimate, and sample.

## What I tried

The plan was a Monte Carlo. Pick a defensible prior for each of Drake's seven factors, draw a million samples from each, multiply them together, look at the resulting distribution of N.

The priors I used, with their published anchors:

- R_star, the Milky Way star formation rate: uniform on 1.5 to 3 stars per year. The Robitaille and Whitney 2010 reanalysis came in lower, around 1 per year, but Drake-style accounting usually includes all stars that have ever existed and may host long-lived civilizations, so a slightly wider range felt right.
- f_p, the fraction of stars with planets: uniform 0.9 to 1.0, following Cassan et al. 2012 and the broader Kepler statistical picture.
- n_e, habitable rocky planets per system: log-uniform 0.1 to 0.6. Petigura 2013 anchored the low end. Bryson 2021 sat at the high end. The two papers used different definitions of "habitable" and "Earth-size", which is exactly why a log-uniform across the disagreement felt fair.
- f_l, fraction where life appears: log-uniform 1e-3 to 1. I am cutting off the floor much higher than Sandberg, Drexler and Ord 2018, who let f_l go down to 1e-30. My reasoning is that life appeared on Earth essentially as soon as the surface cooled, which weakly argues against abiogenesis being arbitrarily hard. This choice matters and I will come back to it.
- f_i, fraction where intelligence evolves: log-uniform 1e-3 to 1.
- f_c, fraction that emit detectable signals: log-uniform 0.01 to 1.
- L, civilization lifetime in years: log-uniform 1e2 to 1e8. The biggest unknown, and as we will see, the one that dominates the answer.

A few notes on those choices. Log-uniform priors are the right default for terms whose order of magnitude is unknown. Multiplying log-uniform random variables produces a distribution whose log is the sum of seven independent uniforms, which is close to Gaussian by the central limit theorem in log space. So the posterior on log10(N) should look roughly Gaussian, with a clean median and a wide spread.

I have been careful not to invent data. Every prior range above traces to a paper I can cite. The one judgment call is f_l, where the literature genuinely disagrees and I picked a middle-ish slice.

The whole simulator fits in about thirty lines of NumPy.

## What happened

The core loop:

```python
R_star = rng.uniform(1.5, 3.0, N)
f_p = rng.uniform(0.9, 1.0, N)
n_e = log_uniform(0.1, 0.6, N)
f_l = log_uniform(1e-3, 1.0, N)
f_i = log_uniform(1e-3, 1.0, N)
f_c = log_uniform(0.01, 1.0, N)
L   = log_uniform(1e2, 1e8, N)
N_civ = R_star * f_p * n_e * f_l * f_i * f_c * L
```

With one million draws, the posterior median came out to 5.1 communicating civilizations in the galaxy. Drake's 1961 estimate was 10. Sixty-five years of astronomy moved the central guess by less than a factor of two.

The 90 percent credible interval is the interesting part: 1.2 × 10⁻³ to 2.2 × 10⁴. That is seven orders of magnitude. The probability that N is less than 1, i.e. we are alone in the galaxy, came out to 0.385. Roughly speaking, more than one chance in three that the Drake terms as I priored them yield no one else at all.

The probability we are in a teeming galaxy with N > 10³ was 0.164. The probability of a Star-Trek-density galaxy with N > 10⁶ was 0.0036.

Then I asked which term keeps the answer uncertain. For each factor, I computed the share of log-variance of N it contributes:

| term | share of var(log10 N) |
|------|-----------------------|
| L (civilization lifetime) | 0.614 |
| f_l (origin of life) | 0.153 |
| f_i (intelligence) | 0.153 |
| f_c (communicates) | 0.068 |
| n_e (habitable planets) | 0.010 |
| R_star | 0.002 |
| f_p | 4 × 10⁻⁵ |

The astronomy terms together account for about 1.2 percent of the uncertainty. R_star and f_p are essentially nailed down. n_e contributes one percent. The biology and sociology terms account for almost everything else, and L alone is responsible for 61 percent of the spread.

That is a hard, clean conclusion. If your goal is to estimate N, building a bigger exoplanet survey is not where the money should go. Drake himself was clear about this in later interviews, but the variance decomposition makes it quantitative.

The other reason this matters: Sandberg, Drexler and Ord (2018) ran a similar exercise with wider priors and got P(alone) around 0.53. My narrower f_l floor (1e-3 instead of 1e-30) pulls that down to 0.385, but "alone" is still the modal outcome. It is hard to write a defensible prior on the Drake terms that does not give substantial mass to N < 1. That fact, more than any specific number, is what sixty-five years of multiplication has taught us.

## What it may mean

The Drake equation has been criticized as scientifically empty because every term can be moved to taste. The Monte Carlo version makes that complaint more precise: the answer is dominated by L, and L is the one term we have absolutely no data on. We have one example of a communicating civilization, and it has been broadcasting for about 100 years out of an unknown total lifetime. We cannot constrain L from N=1.

What the simulation appears to say, on these priors:

- Drake's original median was approximately right by accident. The thing that has changed is not the central estimate but our understanding of why it is uncertain.
- The astronomy terms are essentially done as far as N is concerned. Future exoplanet missions matter for biosignature detection, not for refining N.
- "Are we alone?" is roughly a coin flip with substantial bias toward yes. This is uncomfortable but not crazy.
- Pinning down L would collapse the answer. Nothing else would.

I would not put much weight on the specific number 5.1. A different person making different prior choices could get 0.05 or 500 with no obvious error. The claims that seem to hold up are the variance decomposition and the persistence of the alone-or-empty modal outcome.

## Loose ends

The biggest cheat in this analysis is treating the seven terms as independent. They probably are not. A galaxy where f_l is low might also be one where f_i is low (less metal, less complex chemistry, less time). A planet that easily evolves intelligence might be one that easily extincts it, making L short. Joint priors would change the answer by an amount I have not bothered to estimate. If you had a week, the next thing to do would be to build a small Bayesian network linking f_l, f_i, f_c, L through a hidden "complexity-friendliness" node, and propagate. I suspect it would tighten the posterior considerably while leaving the median near where it is.
