# Verhulst's 1838 logistic fits the US census 1790-1910 within 3.5%, then misses 2020 by 140 million

*Part 83 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Pierre-François Verhulst was a Belgian mathematician writing in
1838, in direct reply to Malthus. Malthus had argued that
populations grow geometrically while food grows arithmetically, so
catastrophe was the long-run default. Verhulst's adjustment was
small but consequential. He kept the exponential at low density and
added one term: as a population approaches the resources available
to it, the per-capita growth rate falls linearly toward zero. In
modern notation,

    dN/dt = r * N * (1 - N/K)

The solution is the S-shaped curve everyone has seen in a biology
textbook. Verhulst fit it to Belgian, French and Russian census
records and named K the population's *limite supérieure*. The paper
was largely ignored. Eighty-two years later Raymond Pearl and Lowell
Reed rediscovered the same equation, fit it to the US census
1790-1910, and published a 1920 forecast of an eventual US ceiling
near 197 million. I wanted to see, on the actual numbers, how well
that early-20th-century fit held up, and where it broke.

## What I tried

The plan had three pieces. First, integrate the logistic ODE
numerically and check it matches the closed form to floating-point
precision. That is housekeeping but worth doing once. Second,
hand-encode the US decennial census from 1790 to 2020 from the
Census Bureau's historical tables, fit (r, K, t_mid) on Pearl and
Reed's 1790-1910 window, and report what the model predicts for the
out-of-sample years 1920-2020. Third, do the same exercise on world
population estimates from 1700 to 2020 (HYDE/McEvedy-style figures
before 1950, UN figures after) and ask whether a single logistic
can carry three centuries of global data.

The fit uses scipy's `curve_fit` on the closed form

    N(t) = K / (1 + exp(-r * (t - t_mid)))

so the three free parameters are intuitive: r is the early
exponential rate, K is the ceiling, and t_mid is the year of
inflection where N = K/2.

I picked the 1790-1910 window on purpose. That is exactly the data
Pearl and Reed had in hand. If our fit agrees with theirs, we know
the optimisation is well-posed. If it does not, something is off in
the data encoding.

## What happened

The US fit on 1790-1910 reproduces Pearl-Reed almost exactly. We
get r = 0.0313 per year, K = 198.5 million, and inflection in 1914.7.
Pearl and Reed reported K ≈ 197.3 million. The difference is less
than one percent. Inside the fit window every census lands within
3.5% of the curve; the worst single point is 1860, presumably
because the 1860s ran into the Civil War.

![US census fit and world residuals](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/83_logistic_fit.png)

The interesting part is what happens after 1910. The Pearl-Reed
curve flattens toward 198 million and stays there. The actual US
does not.

| year | observed (M) | predicted (M) | residual |
|---|---|---|---|
| 1920 | 106.0 | 107.5 | -1.4% |
| 1940 | 132.2 | 136.6 | -3.4% |
| 1960 | 179.3 | 159.8 | +10.9% |
| 1980 | 226.5 | 175.8 | +22.4% |
| 2000 | 281.4 | 185.7 | +34.0% |
| 2020 | 331.4 | 191.4 | +42.2% |

By 2020 the 1910 logistic under-predicts US population by 140
million. The break is not gradual; it sits where the baby boom and
the post-1965 reopening of immigration sit. The model was right
about a regime, not about a destiny.

The way the residuals walk away from zero is worth pausing on. From
1920 to 1940 the curve still tracks observed population to within a
few percent. That is a span of about thirty years past the end of
the fit window in which Pearl and Reed's forecast looked vindicated.
A demographer reading the 1940 census in 1941 would have seen 132
million Americans against a predicted 137 million and concluded
that the logistic was holding up. Only with the 1960 census, two
decades later, does the gap open into double digits in percent
terms. By then K is no longer a ceiling; it is a footnote.

Here is the actual fit, stripped down.

```python
def logistic(t, r, K, t_mid):
    return K / (1.0 + np.exp(-r * (t - t_mid)))

mask = years <= 1910
popt, pcov = curve_fit(logistic, years[mask], pop[mask],
                       p0=[0.03, 200.0, 1910.0])
r_us, K_us, tm_us = popt
print(K_us, np.sqrt(pcov[1, 1]))
# -> 198.5  10.5
```

The world fit is more interesting because it does not really
converge. With population estimates from 1700 (~610 M) to 2020
(~7,795 M), a single (r, K, t_mid) cannot find a stable K: the
optimiser pushes K toward whatever upper bound you give it. The
residuals tell you why. They are large and positive everywhere
before 1900, large and negative through the mid-20th century, then
small and positive again. That is the signature of a model that is
underfitting structure rather than missing noise. Pre-1900 the world
grew slower than any logistic willing to also pass through the
post-1950 figures; the green revolution and fossil-fuel agriculture
appear, on this read, to have *raised K* mid-century.

Inside the 1700-2020 window the world residuals run as large as
+96% in 1700 and -15.6% in 1950. No tweak of a single (r, K) can
remove both signs. That is a structural failure of the model, not a
data problem.

## What it may mean

Two things stand out, both hedged. The logistic appears to be a
genuinely good one-regime descriptor. Where the underlying
assumptions roughly hold (closed population, roughly fixed resource
ceiling, no large discrete shocks), it captures growth curves to a
few percent over more than a century. Pearl and Reed's 1920 US fit
is a clean demonstration. It also matched the data well into the
1930s before the postwar boom pulled the curve away.

What the model does not appear to give us is a long-horizon
forecast in any system where K is itself moving. The US after 1940
and the world over the 18th-20th centuries are both cases where K
is plausibly a function of technology, policy and energy supply.
You can absorb that by letting K be a slow function of time, or by
chaining two or more logistics, but at that point you are doing
phenomenology rather than population biology. The original Verhulst
claim, that resources cap exponential growth, is conserved; the
specific number K becomes a moving target.

There is also a methodological point. The Pearl-Reed forecast
looked good for about 20 years before it visibly broke. That is
useful context for any modern S-curve forecasting (adoption curves,
disease saturation, technology diffusion). The early data may admit
many compatible (r, K) pairs; the late data is what actually pins K
down, and you do not have it yet.

## Loose ends

The world residuals would benefit from a formal runs test on the
sign sequence, which I did not do. The pre-1900 world estimates are
not independent across sources; treating them as if they were makes
the residuals look more decisive than they are. A two-logistic
mixture, or a fit with K(t) as a slow exponential, would probably
collapse the residual structure. The next experiment I would run is
a leave-one-decade-out sensitivity on the US fit, to see at what
year an analyst in real time would have noticed K was rising. That
takes about a day on a laptop.
