# I checked Jakob Bernoulli's 25,550-trial bound from 1713. It holds, with room to spare.

*Part 52 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1713, eight years after Jakob Bernoulli's death, his nephew Nicolaus
finally published the unfinished manuscript Bernoulli had been working
on for two decades. The book was called *Ars Conjectandi*, the art of
conjecturing. Most of it is combinatorics that any modern undergraduate
would recognize. The last part, Part IV, is something else. It contains
the first proof of what we now call the weak law of large numbers, the
statement that if you flip a biased coin enough times, the observed
fraction of heads approaches the true probability. Bernoulli was not
satisfied with a vague convergence claim. He wanted a number. He set
up an urn with 3000 white pebbles and 2000 black pebbles, the true
ratio p = 3/5, and asked how many draws would put the sample ratio
within 1/50 of that with odds 1000:1. After pages of bounding the
binomial tail by hand, he wrote down N >= 25,550.

That bound sat for centuries. Chebyshev's 1867 inequality gave a
general route, but the route for this specific binomial case was
Bernoulli's. I had heard the 25,550 number quoted in lectures and
never actually seen it pressure-tested against simulation.

## What I tried

The plan was three layers. First, simulate Bernoulli(0.6) for N up to
one million, across a dozen seeds, and look at running means. The
trajectories should funnel toward 0.6 as N grows. That checks the
qualitative LLN.

Second, estimate the tail probability P(|X̄_N - p| > epsilon) by
replication. For each N in {100, 300, 1000, 3000, 10000, 30000,
100000}, draw 5000 independent samples of size N from Binomial(N,
0.6), compute the sample mean, and count how many fall outside
[p - epsilon, p + epsilon]. Compare that empirical tail to two
analytic bounds: Chebyshev's p(1-p) / (N epsilon^2), and the CLT
normal approximation. If LLN is real, all three should decay to zero.

Third, the headline check. For Bernoulli's exact parameters, epsilon
= 1/50, target tail 1/1001, search for the smallest N where the
empirical tail falls below the target. If that N is less than 25,550,
Bernoulli was conservative. If it is more, he was wrong. I ran
20,000 replicates per candidate N.

There is one extra test, a control. Replace the Bernoulli samples
with Cauchy(0, 1) draws. The Cauchy distribution has no finite mean.
The LLN should break. If the running mean still settles, my code is
wrong. If it never settles, the finite-variance assumption is doing
real work and Bernoulli's caveats matter.

## What happened

The trajectories funneled in cleanly. By N = 100,000 every seed sat
within 0.003 of 0.6. The standard deviation of X̄_N at each N matched
sqrt(p(1-p)/N) to within about one percent across all sizes I tested,
from N = 10 up to N = 300,000. That is the CLT rate of 1/sqrt(N),
faster than the worst-case Markov bound.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/52_trajectories.png)

The tail comparison was where the surprise lived. Here are the numbers
for epsilon = 0.02:

| N | empirical tail | Chebyshev bound | CLT approx |
|---|---------------:|----------------:|-----------:|
| 100 | 0.758 | 6.00 | 0.683 |
| 1,000 | 0.212 | 0.600 | 0.197 |
| 10,000 | 0.000 | 0.060 | 4.5e-5 |
| 100,000 | 0.000 | 0.006 | 3.9e-38 |

Chebyshev's bound is correct (when finite) but very loose. At N = 100
it gives a probability of 6.00, which is useless because probabilities
cannot exceed 1. At N = 1000 it says the tail is at most 0.6 and the
actual tail is 0.21. The CLT approximation tracks the empirical
numbers closely down to extremely small probabilities.

Now the headline. For Bernoulli's parameters (epsilon = 1/50, target
1/1001), Chebyshev's inequality asks for N >= p(1-p) / (target *
epsilon^2) which works out to N >= 600,600. That is roughly 23 times
larger than Bernoulli's 25,550. In other words, the 1867 generic
bound is much weaker than the 1713 binomial-specific bound for this
problem. Bernoulli's argument used the discrete structure of the
binomial directly; Chebyshev's used only variance.

What does the actual empirical tail look like at N = 25,550? With
20,000 replicates, exactly zero of them fell outside [0.58, 0.62].
So the tail is below 5 * 10^-5, well under the 1/1001 target.
Stepping down through smaller N, the empirical tail crosses 1/1001
somewhere between N = 6,000 and N = 8,000. Bernoulli's bound is
therefore correct but loose by a factor of about 3 to 4. He knew
this. In the original text he writes that the true N is smaller; he
just could not bound it tighter with the methods he had.

```python
# the headline experiment
for N in [4000, 6000, 8000, 10000, 25550]:
    r = np.random.default_rng(99 + N)
    s = r.binomial(N, 0.6, size=20000) / N
    tail = np.mean(np.abs(s - 0.6) > 1/50)
    print(N, tail)
# 4000  0.0096
# 6000  0.0015
# 8000  0.00045
# 10000 0.00010
# 25550 0.00000
```

The Cauchy control behaved as advertised. Eight independent seeds,
N = 200,000 each, and the running means wandered between roughly -10
and +10 at the largest N, with one seed sitting at about +3 and
another at about -4. There is no convergence. The trajectories look
nothing like the Bernoulli plot. This is the standard reminder that
LLN needs a finite mean, which needs at least some control on the
tails.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/52_cauchy_breakdown.png)

## What it may mean

Bernoulli's claim survives. The sample mean of an iid Bernoulli
sequence does converge in probability to p, the fluctuations shrink
at the CLT rate, and his explicit 25,550-trial bound for p = 3/5,
epsilon = 1/50, and odds 1000:1 holds easily. It is loose by a
factor of three or so against the empirical minimum, and tighter
than the generic Chebyshev bound by a factor of more than twenty.
For a result derived without modern calculus and published
posthumously in 1713, that is a strong showing.

There may be a smaller historical lesson here too. We tend to teach
the LLN through Chebyshev because the proof is cleaner. But the first
person to write down a useful sample-size formula did it by exploiting
the specific distribution, and his number is closer to the truth than
the generic 1867 inequality. Distribution-specific bounds beat
generic ones, when you have the patience to derive them.

The Cauchy break should also stay in mind. People sometimes appeal
to the LLN to justify averaging anything noisy. If the noise has
heavy tails (financial returns, network latencies, certain biological
measurements), the sample mean may not be doing what the user thinks
it is doing. Bernoulli's 25,550 only applies when the second moment
exists.

## Loose ends

The search for the empirical minimum N was coarse. A finer grid
would narrow the 6,000-8,000 window further, maybe to 7,000-ish.
The original *Ars Conjectandi* text uses a slightly different
confidence ratio in different passages; I took the 25,550 number
from a standard translation rather than re-deriving from the Latin.
A nice next step would be to reproduce Bernoulli's bound symbolically,
following his exact moves with modern notation, to see where the
factor of three comes from. That is maybe a weekend of work. The
simulation itself takes about 30 seconds on one laptop core.
