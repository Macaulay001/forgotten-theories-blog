# I tested von Mises's 1939 birthday claim with exact arithmetic and 10^5 Monte Carlo trials per group size. 23 people really is enough.

*Part 99 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1939 Richard von Mises published a short paper with the unwieldy title *Über Aufteilungs- und Besetzungswahrscheinlichkeiten*. Inside is the calculation that everyone now calls the birthday paradox. Pick 23 people at random. The probability that two of them share a birthday is just over 1/2.

The number 23 is the surprising part. People hear "23 out of 365" and round it to "rare". The combinatorics work the other way. With 23 people there are 23 * 22 / 2 = 253 pairs, and each pair has a 1/365 chance of matching. The expected number of matching pairs is about 0.69, and once the expectation is that high, the probability of at least one match clears 50%.

I have used this result in arguments for years without ever sitting down and checking it against exact arithmetic and a simulation. I also wanted to test a folk claim I have heard repeated in classes: that the real-world non-uniformity of birthdays (more babies are born in late summer in the US) shifts the crossover meaningfully. Does it.

## What I tried

The exact probability is one line of arithmetic done in log space to avoid overflow:

    P(N) = 1 - 365! / ((365 - N)! * 365^N)

The numerator is the number of ways to pick N distinct birthdays in order, and the denominator is the total number of ordered N-tuples of birthdays. The complement is the probability of no collision.

I evaluated this for N from 2 to 100 by summing logs. At N = 22 the answer is 0.4757. At N = 23 it is 0.5073. So the integer crossover is at N = 23 and the gap is about three percentage points, not a knife's edge.

That is the easy half. The interesting half is the Monte Carlo, for two reasons. First, it independently checks the formula without any algebra. Second, the same code can run a non-uniform variant where birthdays are drawn from a seasonal distribution rather than uniform over 365 days.

For the seasonal distribution I used a single sinusoid approximation taken from the NCHS birth-rate digests summarized in Murphy 2012. US births in the modern data peak around mid-September and trough around early March, with a peak-to-mean swing of roughly 8 to 10%. I encoded that as a daily weight `1 + 0.09 * cos(2*pi*(day - 256)/365)`, normalized to sum to one. It is a coarse model and ignores weekday and holiday structure, but it captures the dominant annual cycle.

For each N from 2 to 100, I drew 100,000 groups of N birthdays under each of the two distributions, counted how many groups contained at least one collision, and recorded the fraction. At 10^5 trials the sampling standard error near P = 0.5 is about 0.0016, which is fine for distinguishing P(22) from P(23) reliably.

## What happened

The exact formula and the uniform Monte Carlo agree to within 0.003 across all N from 2 to 100. The peak deviation was 0.003 at one of the steep middle values of N, and the integer crossover sits at N = 23 in both. At N = 23 the exact value is 0.5073 and Monte Carlo returned 0.5103, a gap of 0.003 which is within two standard errors of zero.

Here is the core of the calculation:

```python
def exact_prob(n):
    log_num = sum(math.log(365 - k) for k in range(n))
    log_den = n * math.log(365)
    return 1.0 - math.exp(log_num - log_den)

def mc_prob(n, weights=None):
    draws = rng.choice(365, size=(TRIALS, n), p=weights)
    sd = np.sort(draws, axis=1)
    has_collision = np.any(sd[:, 1:] == sd[:, :-1], axis=1)
    return has_collision.mean()
```

A small table of the values that matter:

| N  | exact uniform | MC uniform | MC seasonal |
|----|---------------|------------|-------------|
| 5  | 0.0271        | 0.0265     | 0.0269      |
| 10 | 0.1169        | 0.1182     | 0.1175      |
| 23 | 0.5073        | 0.5103     | 0.5107      |
| 30 | 0.7063        | 0.7062     | 0.7080      |
| 50 | 0.9704        | 0.9706     | 0.9710      |
| 70 | 0.9992        | 0.9993     | 0.9993      |

The seasonal numbers are the part I was actually curious about. Theory says any departure from uniform can only raise the collision probability, with equality only when the distribution is exactly uniform. The argument is short: collision probability is a convex function of the daily probabilities, so smoothing them lowers it. The classroom intuition is that bunched birthdays should shift the crossover down to N = 22 or even lower.

That is not what happened. At N = 23 the seasonal Monte Carlo gives 0.5107 versus 0.5103 for uniform, a lift of 0.0004. The lift exists, in the right direction, but it is tiny. The integer crossover does not move. The seasonal P(22) sits at 0.4783, still below 1/2. You would need a much more extreme seasonality, or a few specific densely shared days like January 1 in some birth registries, to bend the crossover.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/99_birthday_curve.png)

The plot makes it visual. The exact curve is a smooth black line. The two Monte Carlo point clouds sit on top of it for all N, with the seasonal points a hair above the uniform points in the steep middle region. By N = 50 the two clouds and the line are indistinguishable at the resolution of the plot.

One number worth keeping. By N = 70 the probability of a shared birthday is 0.9992. By N = 80 it is essentially 0.99998. A small classroom (say 30 people) is already at 71%. The intuition gap is not at N = 23, it is at N = 30 and beyond, where the formula says "near-certain" and people still think "unlikely-ish".

## What it may mean

The textbook result holds up. Nothing exciting there, but the seasonal check is mildly interesting. The folk story that "real birthdays are uneven, so the answer is even smaller than 23" appears to be true in sign and tiny in magnitude. The convexity argument guarantees the sign. The data sets the size, and the size is something like 0.0004 in probability at N = 23 for a realistic US seasonality.

There is a practical residue. Birthday-style collision bounds are how cryptographers reason about hash functions. A k-bit hash has a 50% collision probability after roughly 2^(k/2) random inputs, by the same calculation in a different base. For SHA-256 that is 2^128, which is why no one worries. For a 64-bit hash it is 2^32, about 4 billion, which is small enough to encounter in practice. The same convexity result tells you that any non-uniformity in the hash output makes the attacker's job easier, not harder. The numerical lift at the relevant N might still be tiny, but it is in the wrong direction for the defender.

For social use, the lesson is the one von Mises was already pointing at in 1939. Pairwise counting beats linear intuition. Once you have N people there are N(N-1)/2 pairs, and that quadratic growth is what does the work. The 23 people are not special. The 253 pairs are.

## Loose ends

A more careful seasonal model would use real daily birth counts and would also include weekday effects (US births dip sharply on weekends because of scheduled C-sections and inductions). Both would raise P(N) slightly, and I would guess the combined effect at N = 23 is still under 0.001. A nice next step would be to find the smallest empirical distribution change that does shift the crossover to N = 22, as a sensitivity exercise. With a week of NCHS daily data the whole thing would take an afternoon.
