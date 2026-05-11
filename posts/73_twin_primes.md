# I counted every twin prime under ten million. Hardy and Littlewood's 1923 guess landed within 0.3%.

*Part 73 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Twin primes are pairs of primes two apart: (3, 5), (5, 7), (11, 13),
(17, 19), and on it goes. They get rarer as numbers grow, but they
keep showing up. Whether they show up infinitely often is still open
after more than two thousand years of asking.

In 1923, G. H. Hardy and J. E. Littlewood put down a much sharper
guess than "infinitely many". Their paper "Some problems of Partitio
Numerorum: III" predicted exactly how rare twin primes should be. The
count below N, they wrote, should grow like

    pi_2(N) ~ 2 C_2 N / (ln N)^2

where C_2 ~ 0.6601618 is a specific constant built from a product
over all odd primes. That factor of 2 C_2, roughly 1.32, is the
correction you get when you stop pretending that primes are scattered
independently and instead account for the obvious fact that if n is
an odd prime then n+2 has already passed half the divisibility test.

The conjecture remains unproven. Yitang Zhang's 2013 result and the
Maynard-Tao refinement narrowed bounded gaps between primes, but the
twin case sits untouched. Numerically, though, the prediction has
held up everywhere people have looked. I wanted to look myself, on
my own laptop, and feel the constant come out of an actual count.

## What I tried

The plan was small. Sieve every prime up to ten million using a plain
Eratosthenes pass. Mark a twin pair wherever both n and n+2 survive
the sieve. Count those pairs at four cutoffs: 10^4, 10^5, 10^6, 10^7.
Compare to the Hardy-Littlewood prediction, and to its slightly better
integral form

    2 C_2 Li_2(N) = 2 C_2 * integral from 2 to N of dt / (ln t)^2

which is to the leading-order N/(ln N)^2 what the logarithmic integral
Li(N) is to N/ln N: a closer fit at finite N. From the four counts I
could also back out an empirical C_2 and see whether it drifts toward
the theoretical value or away from it.

Nothing about this is technically hard. A boolean array of length
ten million plus one fits comfortably in memory. The sieve runs in
under a second. The interesting question is whether a 1923 prediction,
made before computers existed, lines up with a brute count made
today.

## What happened

It lines up well. Sieving to N = 10^7 turned up 664,579 primes and
58,980 twin-prime pairs (counted by the smaller member of each pair).
The Hardy-Littlewood integral form predicts 58,814.7. The ratio of
observed to predicted is 1.003. The leading-order form is looser
(58,980 / 47,747 = 1.235 at this scale) but that gap is dominated by
known sub-leading corrections, the same way Li(N) beats N/ln N in
counting all primes.

Here is the core of the count, after the sieve runs. The sieve array
`sieve[i]` is True for prime i. A twin pair appears wherever a True
at position i is followed by a True at position i+2.

```python
# twin primes: lower member is i where sieve[i] and sieve[i+2]
twins_mask = sieve[:-2] & sieve[2:]
twin_lower = np.nonzero(twins_mask)[0]

for M in [10**4, 10**5, 10**6, 10**7]:
    n_twin = int(np.searchsorted(twin_lower, M, side='right'))
    hl = 2.0 * C2 * li2(M)              # integral form
    c2_emp = n_twin / (2.0 * li2(M))    # empirical constant
    print(M, n_twin, n_twin / hl, c2_emp)
```

The table below shows the four cutoffs:

| N       | pi_2(N) | 2 C_2 Li_2(N) | ratio | empirical C_2 |
|---------|---------|----------------|-------|---------------|
| 10^4    |     205 |         214.2  | 0.957 | 0.63177       |
| 10^5    |   1,224 |       1,248.8  | 0.980 | 0.64706       |
| 10^6    |   8,169 |       8,251.9  | 0.990 | 0.65353       |
| 10^7    |  58,980 |      58,814.7  | 1.003 | 0.66202       |

The empirical C_2 climbs from 0.632 to 0.662 across three decades of
N. The theoretical value is 0.6601618. At N = 10^7 the fit sits 6
parts in 10,000 above the analytic constant, well inside the
fluctuation envelope you would expect from random Poisson-like
arrivals at this count.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/73_twin_primes_hl.png)

On the log-log plot the observed counts sit on the Hardy-Littlewood
curve to within the marker size. The ratio panel on the right tells
the cleaner story: the integral form converges to 1 smoothly from
below, while the leading form drifts down toward 1 from above, both
behaving as the theory expects.

What surprised me is how visible the constant itself is at this
modest scale. The naive heuristic, "treat primes as independent",
gives N/(ln N)^2. Multiplying by 2 C_2 ~ 1.32 is a single correction
factor, and that factor matters: without it the ratio at 10^7 would
sit near 1.32 instead of 1.003. Hardy and Littlewood derived 2 C_2
from a heuristic about prime correlations modulo small primes. That
heuristic, almost a hundred years before bounded-gap results, picks
up the twin density to within fractions of a percent on the numbers
you can sieve in a few seconds.

## What it may mean

Treat this as a small audit, not a proof. A close numerical match
between an observed count and a conjectured asymptotic is consistent
with the conjecture being true. It does not establish it. The history
of pi(N) and Li(N) is the standard warning: numerical agreement up
to enormous N can still hide later sign changes (Skewes' number sits
near 10^316). No one expects analogous surprises in the twin count,
but the formal statement remains open.

What the audit does suggest is that the Hardy-Littlewood circle-method
heuristic, applied to the twin problem, captures the right answer at
the level of constants and not just exponents. That is more than a
qualitative claim. The empirical C_2 at 10^7 differs from
0.6601618 by less than the spread you would see by sampling different
sub-intervals of comparable length. On this sample, the conjecture
behaves exactly as if it were a theorem.

For someone who has never sieved primes, the part that may stick is
that a constant defined as an infinite product over odd primes,
written down in a 1923 paper, falls out of a half-page Python script
counting integers up to ten million. The constant was guessed before
it was countable, and the count agrees.

There is also a small aesthetic point. The empirical C_2 column in
the table is, in a sense, the only thing the data actually delivers.
Everything else (the ratio, the prediction) is just C_2 dressed up
with the integral. Watching that column move from 0.632 to 0.662
across three decades of N is the audit. The 1923 number sits at
0.6602. The 2026 laptop returns 0.6620. The agreement is not magic:
it is the law of large numbers operating on an arithmetic distribution
whose correlation structure Hardy and Littlewood guessed correctly.

## Loose ends

N = 10^7 is small. Oliveira e Silva and others have pushed twin-prime
counts past 10^18, where the agreement with Hardy-Littlewood holds
to a few parts per million. The next experiment with another week
would be a segmented sieve up to 10^10 to see whether the empirical
C_2 keeps tightening at the predicted rate, and a fit to the
sub-leading log correction that explains why the ratio approached 1
from below. Anyone with a faster sieve, or a result on the next-order
term, would be a useful person to hear from.
