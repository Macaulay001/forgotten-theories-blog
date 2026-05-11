# I re-ran Fisher's chi-square check on Mendel's 1866 pea data. The fit really is in the bottom 5%.

*Part 41 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1866, Gregor Mendel published counts from a decade of crosses
between pea plants. He claimed that when you cross two hybrids, the
next generation splits in a 3 to 1 ratio of dominant to recessive
forms. He reported seven different traits (seed shape, seed color,
flower color, pod shape, pod color, flower position, stem length)
across sample sizes from 580 to 8,023 plants, and every ratio came out
close to 3:1. The paper was ignored for 34 years. When it was
rediscovered in 1900, it became the foundation of genetics. Then in
1936, R.A. Fisher reanalysed the numbers and made a quieter, stranger
claim: the data fit the theory so well that no real experimenter
should expect to get them. People have been arguing about that
sentence for ninety years.

I wanted to put a number on the argument.

## What I tried

I hand-encoded the seven monohybrid totals from Mendel's 1866 paper
into a CSV. There is no ambiguity in the source: the counts are in
table form, in the paper itself, reproduced in every textbook since.
Then I did two things.

First, the standard test. For each experiment, compute the chi-square
statistic against the 3:1 expectation. One degree of freedom per
experiment. If Mendel's model is right, each chi-square should be
small but not absurdly small; the expected value is 1.0 per
experiment.

Second, the Fisher check. Sum the seven chi-squares. Under the null
hypothesis (a true 3:1 process, sampled honestly), that sum follows a
chi-square distribution with 7 degrees of freedom, mean 7. If the
total comes out way below 7, the data are suspiciously tidy. If it
comes out way above, the model is wrong. Fisher computed roughly 2.1
in 1936 and noted that the probability of getting a fit that close or
closer is about 4 in 100.

I wanted to see this with my own eyes rather than trust the textbook
retelling. So I ran a Monte Carlo. Fifty thousand simulated replicates
of Mendel's whole experimental program. For each replicate, I drew
seven binomial samples (n equal to Mendel's actual sample size, p =
0.75 dominant), computed the seven chi-squares, summed them. That
gives an empirical distribution of "what should this number look like
if a real geneticist ran Mendel's experiments today, with no
fudging".

```python
# 50,000 replicates of Mendel's seven experiments, summed chi-square
sim_total = np.zeros(N_SIM)
for n in sample_sizes:
    dom = rng.binomial(n, 0.75, size=N_SIM)
    rec = n - dom
    sim_total += (dom - 0.75*n)**2 / (0.75*n) + (rec - 0.25*n)**2 / (0.25*n)

mendel_total = 2.139
frac_at_or_below = (sim_total <= mendel_total).mean()  # = 0.0481
```

That single number, 0.0481, is the whole story.

## What happened

The per-experiment fits are all clean. Every chi-square sits well
below the 5% significance threshold of 3.84. The seed-color experiment
with 8,023 plants gives an observed ratio of 3.009 and a chi-square of
0.015, which is genuinely beautiful. The smallest experiment (pod
color, 580 plants) gives 2.816 with chi-square 0.45. Nothing is
significant. Nothing comes close to significant.

![Mendel's seven monohybrid ratios versus the 3:1 expectation](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/41_ratios.png)

The total is where it gets interesting.

| | value |
|---|---:|
| Mendel's combined chi-square (7 experiments) | 2.139 |
| Theoretical mean of chi^2(7) | 7.000 |
| Simulated mean across 50,000 replicates | 6.996 |
| Simulated median | 6.355 |
| Fraction of simulations with combined chi^2 ≤ 2.139 | 0.0481 |

So my simulation confirms the textbook number, computed independently
from a different angle. About 1 in 21 fresh replications of Mendel's
program, run honestly, would produce a fit this tight or tighter. The
expected combined chi-square is 7. He got 2.1. That is not a wildly
impossible outcome (it is roughly a two-sigma event), but it is
unusual, and reasonable people have noticed.

![Where Mendel's combined chi-square sits on the Monte Carlo null](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/41_monte_carlo.png)

The histogram makes the puzzle concrete. The simulated distribution
peaks around 6, has a long right tail, and a thin left tail. Mendel's
red line sits in that thin left tail, where roughly 5% of honest
replicates land.

This is what Fisher was pointing at. Not that the biology was wrong
(it is not), and not that Mendel was a fraud (Fisher was careful not
to say that), but that the data are tidier than the sampling theory
predicts.

Several things might cause that. The most popular hypothesis since
the 1960s is that Mendel's assistant (he had a gardener who helped
with the counting) may have unconsciously classified ambiguous
seedlings in the direction Mendel expected. Wrinkled-versus-round
seeds are not always clear at the borderline. A 1% bias in classifying
maybe-wrinkled seeds toward the round category would tighten the
ratios exactly the way we see. A second hypothesis is stopping bias:
counting until the ratio looked right and then declaring the
experiment finished. A third is selection of which experiments to
publish. Mendel ran many crosses; he tabulated seven trait pairs.

I cannot distinguish among these from the published totals alone.
None of them require Mendel to have lied. All of them are consistent
with the chi-square pattern.

The per-experiment percentile plot makes the texture visible:
no single experiment is suspicious on its own. The total only looks
strange when you pool. That is the Fisher trick, and it is the right
one. Each individual fit is unremarkable. The collective fit, across
seven experiments, is what bends the curve.

![Per-experiment percentile of Mendel's chi-square on the simulated null](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/41_percentiles.png)

## What it may mean

The biology is fine. Mendel's 3:1 ratio is one of the most replicated
findings in the history of science; modern undergraduate labs
reproduce it every semester. Nothing in this analysis questions the
model.

What is at stake is something narrower: the early-data-collection
norms of a 19th-century friar working alone in a garden. The pattern
we see (chi-square at the 5th percentile, with no single experiment
flagged) is exactly what you would expect from an honest investigator
whose helper unconsciously broke ties in the expected direction. It is
also what you would expect from selective stopping. It is also what
you would expect, with 5% probability, from pure luck.

In this sample (seven monohybrid experiments, pooled), Mendel's data
appear closer to the model than random sampling would typically
deliver. The effect is real but small. Calling it fraud is unsupported.
Calling it perfectly clean is also unsupported. The fairest read may
be that some unconscious sharpening occurred at the borderline cases,
which fits the historical record of how the work was done and which
does not in any way weaken the biology.

A more interesting question, to me, is what we should learn about
reading single-author data from the 1800s. When the ratios are this
clean at this sample size, the right move is the one Fisher made: pool
the chi-squares and ask where the total sits on a null distribution.
The Monte Carlo here costs about 0.1 seconds and tells you the answer
without any analytic trick. It is the cheapest sanity check there is.

## Loose ends

I only re-ran the seven monohybrid totals. Mendel also reported
dihybrid (9:3:3:1) and trihybrid crosses, and Fisher pulled some of
those into his combined statistic too. Adding them would tighten the
percentile estimate. The simulation also assumes pure binomial
sampling; real F2 plants have small linkage effects at some loci and
some viability differences, both of which would slightly increase the
expected chi-square and push Mendel's value into an even thinner tail.
If anyone has a clean digitization of the dihybrid counts plant by
plant, please get in touch. An afternoon of work would extend this.
