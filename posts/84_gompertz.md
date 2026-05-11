# I refit Gompertz's 1825 mortality law on a US 1900 life table. One slope explains 97% of the log-hazard.

*Part 84 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Benjamin Gompertz read a paper to the Royal Society in June 1825
titled "On the Nature of the Function Expressive of the Law of Human
Mortality". He was an actuary, not an academic biologist, and he
wanted a single function that an insurance house could use to price
annuities at any adult age. Looking at the life tables available to
him (Northampton, Carlisle, and a few Swedish parishes), he noticed
that the hazard of dying did not just rise with age. It rose in a
way that, plotted on a log scale, looked very nearly linear. He
proposed that the force of mortality follows a geometric progression:

    mu(t) = alpha * exp(beta * t)

Two parameters. One sets the floor at young adult ages, the other
sets how fast the floor multiplies. For most of the 19th century this
was the law of mortality. It was forgotten only in the sense that
modern hazard models bury it inside Cox regressions, Weibull baselines,
and gerontology arguments about whether the curve flattens at
extreme ages. The original claim sits underneath all of that.

I wanted to know how well it still holds if you take an old life table
at face value and fit nothing fancier than a straight line.

## What I tried

The plan was small. Pick a historical life table that is not so old
the data is famously bad (Northampton was full of mistakes) and not
so recent that it has been graduated to within an inch of its life.
I picked the United States 1900-1902 life table, total persons, as
published by J. W. Glover (1921) for the US Bureau of the Census. It
is one of the first proper US national life tables, and the underlying
mortality data is from registration states.

I hand-encoded qx (the probability of dying within one year of age x)
at five-year intervals from age 30 to age 95. Fourteen points. I did
not transcribe the whole table because I wanted the test to be
something a reader can re-key in fifteen minutes. The source row at
age 95 is mildly stylised. The Glover tables thin out and use
graduation at the oldest ages, so I rounded to a value consistent
with the published trend.

Converting qx to the continuous hazard mu(t) uses a standard piecewise
assumption: if you treat the hazard as constant within a year of age,
then mu(t + 0.5) is roughly -log(1 - qx). I evaluated mu at the
mid-year of each age cell.

Then three fits.

First, Gompertz by ordinary least squares on log(mu) vs age. The law
says log(mu) = log(alpha) + beta * t, so a straight line on a log-y
plot is the whole prediction.

Second, Makeham's 1860 extension, mu(t) = alpha * exp(beta * t) + c,
which adds a constant background hazard. Makeham argued that you
need this term because some causes of death (accidents, infections in
the 19th century) do not scale with age. It is a nonlinear fit in
the parameters once c is in the mix, so I used scipy curve_fit.

Third, a Gompertz fit truncated to age 80 and then extrapolated out
to 95, to see whether the slope you would extract from "young adult"
data over-predicts mortality at the oldest ages. This is the cheap
proxy for the Vaupel late-life plateau question.

## What happened

The Gompertz fit on the full 30-95 range gave beta = 0.0613 per year
and alpha = 9.75e-4. R^2 in log space is 0.967. The mortality rate
doubling time, log(2) / beta, comes out to 11.3 years. Modern
populations report MRDT closer to 7-8 years, and the older table
gives a flatter slope partly because background causes of death are
large in 1900 and inflate the apparent intercept when you do not
model them separately.

That is exactly what Makeham predicted in 1860.

When I added the constant c, the Makeham fit gave alpha = 2.33e-4,
beta = 0.0793 per year, and c = 3.91e-3 per year. The senescence
slope steepens to a more modern-looking value, and the linear-scale
R^2 rises to 0.998. About 0.4% per year of age-independent baseline
hazard is absorbing what was masquerading as a higher floor in the
pure Gompertz fit.

![Gompertz and Makeham on US 1900-1902 life table](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/84_gompertz_logmu.png)

Here is the core of the fit. No imports, no boilerplate. T is the
mid-interval age vector, MU is the hazard vector.

```python
# Gompertz: linear OLS in log space
logmu = np.log(MU)
A = np.vstack([np.ones_like(T), T]).T
(log_alpha, beta), *_ = np.linalg.lstsq(A, logmu, rcond=None)
alpha = np.exp(log_alpha)
mrdt = np.log(2.0) / beta

# Makeham: nonlinear LS in linear mu space
def makeham(t, a, b, c):
    return a * np.exp(b * t) + c
popt, _ = curve_fit(makeham, T, MU, p0=(alpha, beta, 1e-3))
```

A small table makes the slope-vs-intercept trade clearer.

| fit | alpha | beta (per yr) | c (per yr) | R^2 |
|---|---|---|---|---|
| Gompertz, 30-95, log OLS | 9.75e-4 | 0.0613 | - | 0.967 (log) |
| Gompertz, 30-80, log OLS | 1.49e-3 | 0.0526 | - | extrapolates above data at 90-95 |
| Makeham, 30-95, NLS | 2.33e-4 | 0.0793 | 3.91e-3 | 0.998 (linear) |

The truncated Gompertz is the interesting one. Fit on age 30 to 80
only, the slope drops to beta = 0.0526 per year (MRDT 13.2 years).
Extrapolate that line out to 95 and it sits noticeably above the
observed mortality at 90 and 95 in the table. The exponential
projected from middle age over-predicts the late-life hazard. That
is the direction Vaupel and colleagues reported in their late-life
plateau work in the late 1990s, though resolving it properly needs
extreme-age data this little 14-point table does not have.

For comparison the figure also shows a Kannisto-style logistic
curve, mu(t) = a * exp(b * t) / (1 + a * exp(b * t) / M), with M
fixed at a hypothetical plateau ceiling near 0.7 per year. It bends
over the Gompertz line at the oldest ages. I am not fitting that
form to the data here. I am drawing it as a comparator so the
reader can see what the plateau hypothesis looks like geometrically.

## What it may mean

Gompertz published two years of life table work and got a functional
form that, two centuries later, still passes a least-squares test on
a national life table to within a few percent of the variance. That
is unusual. Most quantitative laws from 1825 either turn out to be
local approximations (Hooke), special cases (Newton's cooling), or
just wrong (most of phlogiston-era chemistry). Gompertz's law sits
closer to Kepler in that respect: an empirical regularity that the
later theory has to recover, not contradict.

The hedged version of the conclusion may be the right one. The
straight line in log space is real on this sample, and on most adult
life tables anyone has fit since. The slope beta varies by population
and era: roughly 0.05 to 0.10 per year, which is a doubling time of
seven to fourteen years. Whether the line keeps going at extreme
ages, or bends over into a plateau, is still an active debate in
biodemography. Barbi et al. (2018) reported a plateau in Italian
centenarians; Newman (2018) argued the apparent plateau is age
misreporting plus selection. The historical table I used here cannot
arbitrate that, and I am not claiming it does.

What it can say is that the simple two-parameter law does the
heaviest lifting. The third parameter c (Makeham) is a refinement at
younger adult ages. The plateau, if real, is a second-order
refinement at the very oldest ages. The geometric progression
Gompertz wrote down in 1825 is the first-order story.

## Loose ends

The big caveat is the hand-encoded table. Fourteen points at five-year
intervals is enough to see the line and the curvature, but it is not
enough to resolve a plateau against the alternative of slow Gompertz
deceleration. With another week I would re-fit against the full
Human Mortality Database (HMD) extracts for several countries and
decades, look at how beta has drifted from 1900 to 2020 (the
literature consensus is that it has steepened modestly), and test
the plateau hypothesis on populations with verified extreme ages.
You can replicate the fit in this post in about a minute on a laptop;
the harder version with HMD takes a weekend.
