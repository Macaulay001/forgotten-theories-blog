# I checked Newcomb's 1881 first-digit law on eight datasets. It held on six, and failed on the two controls where it was supposed to fail.

*Part 31 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1881, the astronomer Simon Newcomb published a one-page note in the *American Journal of Mathematics* about a strange thing he had noticed in his library. The logarithm tables on his shelf were dirtier and more dog-eared in the front. The pages for numbers starting with 1 were worn out. The pages for numbers starting with 9 were almost pristine. He took the observation seriously enough to derive a formula: the probability that a "natural number" begins with digit d should be log10(1 + 1/d). That gives about 30.1% for 1, dropping to 4.6% for 9.

The note was forgotten for fifty-seven years. In 1938 a physicist at General Electric named Frank Benford rediscovered the rule, this time with data. He tabulated 20,229 numbers from rivers, populations, molecular weights, baseball stats, and an issue of *Reader's Digest*. The match to log10(1 + 1/d) was good enough that the rule now bears his name and not Newcomb's, which feels slightly unfair.

I had used Benford's law informally for years (to spot suspicious-looking accounting tables) without ever actually fitting it. So I spent an evening doing it on eight datasets.

## What I tried

The test is short. For each dataset, take the absolute value, strip zeros, find the first non-zero digit, and count how often each of 1 through 9 appears. Compare against the Newcomb prediction with a chi-square (eight degrees of freedom) and a total variation distance. That's it.

The interesting part is what to test it on. I wanted a mix:

- Fibonacci numbers from 1 to 1000. These are a known theorem case (the logs are equidistributed mod 1 by Weyl's 1916 result), so they should pass trivially. Sanity check.
- Powers of 2, exponents 1 to 1000. Same theorem applies.
- World city populations. I pulled a curated list of about 140 cities spanning 21,000 (a small town) up to 37 million (Tokyo). Five orders of magnitude.
- River lengths in kilometres. A list of 153 major and minor rivers from 60 km to 6650 km.
- Physical constants. About 67 numbers, mixing tiny things like Planck's constant with big things like Avogadro's number and the Sun's mass. Twenty-plus orders of magnitude.
- Synthetic stock prices drawn from a log-normal with mu = 4, sigma = 1.5, sample size 5000. A stand-in for "prices that move multiplicatively".
- A uniform U[1, 1e6] sample, n = 5000. This is the first control. It should not match Benford.
- A normal(100, 15) sample, n = 5000. This is the second control. It really should not match Benford.

The Fibonacci and powers-of-2 cases have so many digits per number that I had to take the leading character of the string representation rather than try to compute log10 on a 200-digit integer. The other cases use floating point with the standard log-floor trick.

```python
import numpy as np
benford = np.array([np.log10(1 + 1/d) for d in range(1, 10)])

def first_digit(x):
    x = np.abs(x[(x > 0) & np.isfinite(x)])
    exps = np.floor(np.log10(x))
    return np.floor(x / 10**exps).astype(int)

d = first_digit(populations)
counts = np.array([(d == k).sum() for k in range(1, 10)])
expected = benford * counts.sum()
chi2 = ((counts - expected)**2 / expected).sum()
```

Eight datasets, one chi-square each, one figure with eight panels. About forty lines of Python including the embedded city and river lists.

## What happened

Six of the eight datasets passed at alpha = 0.05. The two that failed were the two controls. That is exactly what the theory predicts.

| Dataset | n | chi-square | p-value | TV distance |
|---|---:|---:|---:|---:|
| Fibonacci (1..1000) | 1000 | 0.17 | 1.00 | 0.004 |
| Powers of 2 (1..1000) | 1000 | 0.16 | 1.00 | 0.003 |
| Stock prices (log-normal) | 5000 | 4.95 | 0.76 | 0.011 |
| World city populations | 137 | 8.80 | 0.36 | 0.091 |
| River lengths (km) | 153 | 7.38 | 0.50 | 0.098 |
| Physical constants | 67 | 11.55 | 0.17 | 0.166 |
| Uniform[1, 1e6] | 5000 | 2017 | < 1e-300 | 0.270 |
| Normal(100, 15) | 5000 | 8930 | < 1e-300 | 0.523 |

The Fibonacci and powers-of-2 numbers were the closest. Total variation distance from Newcomb's prediction came out at 0.004 and 0.003 respectively. That is at the level of rounding. These were the cheating cases. They had to work.

What surprised me a little was how well the curated lists did. The river-length list is small (153 entries) and almost certainly biased toward rivers that are big and named enough to make a Wikipedia table. The city list cuts off below 21,000 inhabitants, which is a sharp lower bound that should bend the curve. Both still passed the chi-square test with p around 0.4 to 0.5. Total variation was around 0.09 to 0.10, which is noticeable on the figure but not statistically distinguishable from Benford at this sample size.

The physical constants gave the most interesting outcome. With only 67 numbers, the test has low power, but the empirical distribution still landed within total variation 0.17 of Newcomb's prediction. The first digit was 1 about 38% of the time (Newcomb says 30%) and 9 about 1.5% (Newcomb says 4.6%). The deviation is partly because physical constants cluster around values like c (begins with 2 or 3 depending on units), e and h (begins with 1 or 6), and astronomical masses (begin with 1, 5, 6, 9). I find this funny: the dataset where Newcomb's mechanism should be cleanest, because the units are arbitrary and the magnitudes span thirty orders, is also where the human-selection effect is loudest.

The uniform control failed by a chi-square of 2017. With n = 5000, that p-value is so small that scipy returns it as effectively zero. The reason is that the uniform distribution on [1, 1e6] puts about 90% of its mass in the top decade (100000 to 1000000), so the leading digit there is whatever digit you happen to land on in that interval. The result was a fairly flat distribution across digits 1 through 9, which is the opposite of Newcomb's curve.

The normal(100, 15) control failed harder, by a chi-square of 8930. About 90% of that sample is between 70 and 130, so "1" shows up as the leading digit about 70% of the time. Newcomb says 30%. Three sigma is not enough range for the law to take hold.

## What it may mean

The folk version of Benford's law says "real-world data follows the rule". On this sample, that is roughly right but not quite. The law fits when a dataset spans several decades of magnitude with no preferred scale. The river lengths span two decades, the populations span four, the constants span thirty. All three pass. The two controls span one decade or less, with mass concentrated near a single value. Both fail.

This matches Theodore Hill's 1995 result that Benford emerges as the unique scale-invariant distribution on leading digits. If you can re-scale a dataset (kilometres to miles, dollars to euros) and the distribution of leading digits should not change, then Newcomb's formula is the only option. Datasets concentrated near a single scale do not satisfy that invariance, so they do not need to obey the law.

For practical use, this matters because Benford is often deployed as a fraud or anomaly detector. The IRS uses it for tax-return audits. Election forensics applies it to precinct counts, though precinct counts often lack the magnitude range the law needs (a precinct with 1000 voters has all four-digit counts in the same decade, and the leading-digit test misfires). Accounting fraud detection (Nigrini 1996) works because invoice amounts span many decades. Election forensics is contested for the same reason it would fail on my normal(100, 15) sample.

The Greek government's pre-2010 macroeconomic figures, reportedly, deviated from Benford in a way that was visible after the fact. I have not re-checked that, but the mechanism (a manipulated dataset where humans pick numbers, and humans avoid "1" because it feels too round) is plausible.

## Loose ends

Eight datasets is not a survey. What I would do with another week: pull the actual NYSE intraday tick data for a few hundred securities and test whether real prices behave like the log-normal synthetic, scrape OpenStreetMap for every named river over 50 km, and run the test on second-digit and joint first-two-digit distributions to see whether power increases. The conditional digit law (Pinkham 1961) gives finer discrimination than the leading digit alone and might catch deviations the chi-square misses on n = 67. None of this is expensive. The full pipeline above runs in about thirty seconds on a laptop, and you can replicate it from the embedded code in roughly ten minutes.
