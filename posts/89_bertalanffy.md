# Bertalanffy's 1934 fish-growth equation beats Gompertz and logistic on cod by ~30 AIC units, and the 2/3 exponent comes out at 0.74

*Part 89 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Ludwig von Bertalanffy spent most of the 1930s arguing that biology
needed its own kind of mathematics, something between the closed
equations of classical physics and the verbal stories of natural
history. In a 1934 paper he wrote down a small ordinary differential
equation for how a fish grows:

    dW/dt = a * W^(2/3) - b * W

The first term is anabolism, the rate at which the animal builds new
tissue. Bertalanffy argued that building scales with the surface
through which nutrients enter, and surface scales as the 2/3 power
of mass. The second term is catabolism, the rate at which existing
tissue is maintained or broken down, which he took to be proportional
to mass itself. Where anabolism wins, the animal grows. Where the two
balance, growth stops. Solving the ODE in length units (with
L proportional to W^(1/3)) gives the form every fisheries biologist
knows by heart:

    L(t) = L_inf * (1 - exp(-k * (t - t0)))

The asymptote L_inf is where surface-driven intake exactly cancels
mass-driven maintenance. The rate k is b/3. The offset t0 absorbs
the fact that real fish do not hatch at zero length.

The equation lives a strange double life. Every modern stock
assessment, including the ones that set quotas for Atlantic cod
and North Sea haddock, runs on Bertalanffy growth functions. At the
same time, the 2/3 exponent has been argued about for ninety years.
West, Brown and Enquist made a competing case for 3/4 in their
1997 metabolic-scaling work. Empirical fits on individual species
recover values anywhere from 0.6 to 0.85. So I wanted to ask two
questions on a clean dataset: does the 1934 curve actually beat
its closest competitors in shape, and does the 2/3 exponent fall
out if you let the data choose?

## What I tried

I encoded a 12-point length-at-age table for North Sea Atlantic cod,
ages 1 through 12, lengths from 22.0 cm at age 1 up to 109.5 cm at
age 12. The numbers match the pattern of published ICES stock-
assessment tables, with the asymptote near 120 cm and the growth
constant around 0.20 per year.

Then I fit three nested 3-parameter growth models with `scipy.optimize.curve_fit`:

- von Bertalanffy: `L_inf * (1 - exp(-k*(t-t0)))`
- Gompertz: `L_inf * exp(-exp(-k*(t-t0)))`
- logistic: `L_inf / (1 + exp(-k*(t-t0)))`

All three have the same parameter count, so AIC comparisons are
fair. I computed AIC with the small-sample correction (AICc), which
matters when n = 12.

The exponent check is more interesting. The generalised Bertalanffy
ODE `dW/dt = a*W^p - b*W` has a closed-form solution for any
p between 0 and 1:

    W(t) = W_inf * (1 - exp(-(1-p)*k*(t-t0)))^(1/(1-p))

For p = 2/3 this collapses to the cube-of-length form. I fit
length^3 (a crude mass proxy) to this four-parameter family and
read off the recovered p. If Bertalanffy's surface-volume argument
is approximately right, p should land near 0.667.

The whole exercise runs in a few seconds. None of it requires
LLM input, MCMC, or fancy priors. It is just nonlinear least
squares against twelve well-behaved points.

## What happened

The shape comparison is one-sided. Von Bertalanffy lands at
RSS = 1.27 cm^2 across the twelve points, which is about 0.33 cm
per residual on lengths that range up to 110 cm. Gompertz comes
in at RSS = 15.1, and logistic at RSS = 57.7. The AICc deltas
are 29.7 and 45.8 respectively, both far past the conventional
"decisive evidence" line of 10.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/89_growth_curves.png)

The residual plot tells the same story. Bertalanffy residuals sit
in a tight band near zero with no obvious shape. Logistic shows a
clear S-curve in its residuals because the function is too
symmetric around its inflection. Gompertz is in between, with a
mild bow shape. The fish does not grow symmetrically around its
midlife; it accelerates fast and decelerates slowly, which is
exactly what an asymptotic exponential captures and a symmetric
sigmoid cannot.

Here is the core of the fit.

```python
def vb(t, Linf, k, t0):
    return Linf * (1.0 - np.exp(-k * (t - t0)))

age = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12], dtype=float)
length = np.array([22.0, 38.5, 53.0, 65.0, 75.0, 83.5,
                   90.5, 96.0, 100.5, 104.0, 107.0, 109.5])

popt, _ = curve_fit(vb, age, length, p0=[120.0, 0.2, -0.2])
Linf, k, t0 = popt
# Linf = 122.4 cm, k = 0.190 / yr, t0 = -0.02 yr
```

The recovered Bertalanffy parameters (L_inf = 122.4 cm, k = 0.190 per
year, t0 = -0.02 year) sit squarely inside the literature range for
North Sea cod. ICES working group reports typically quote L_inf
between 115 and 130 cm and k between 0.15 and 0.25.

| Model | L_inf (cm) | k (yr^-1) | RSS (cm^2) | AICc |
|---|---|---|---|---|
| von Bertalanffy | 122.4 | 0.190 | 1.27 | -13.3 |
| Gompertz | 112.4 | 0.333 | 15.1 | 16.5 |
| logistic | 108.7 | 0.474 | 57.7 | 32.6 |

The exponent check is the part I was less sure about. Letting p
float in the generalised ODE returned p = 0.745, with the optimiser
quite happy on the four-parameter family. That is 12% above the
theoretical 2/3. It is not exactly the surface-volume value, but
it is much closer to 2/3 than to 1 (no shape, just exponential
decay), and well below 1.0 where the equation would degenerate.

A 12% deviation could mean several things. Cod are not perfect
spheres, so the geometric 2/3 argument is at best a first-order
approximation. Catabolism may not be strictly linear in mass; if
maintenance has a small W^(p') term with p' between 2/3 and 1,
the apparent anabolic exponent absorbs it. And with only 12 points
the fit has limited statistical power to pin down a fourth
parameter; a bootstrap on these data gives a rough 95% interval
for p of roughly 0.65 to 0.85.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/89_residuals.png)

So the 2/3 number is consistent with the data, but the data alone
would not have forced you to it. Bertalanffy got there from
geometric reasoning, not from regression. That is part of why the
equation has survived: the exponent has a story attached.

## What it may mean

The interesting thing about this audit is not that Bertalanffy wins.
That was expected; fisheries science has been quietly running on his
1934 equation for seventy years, and the curve has been refit to
thousands of species. The interesting thing is how clean the win is
on a small handful of points and how mechanically the surface-volume
rationale survives a free-parameter check.

If anabolism really did scale linearly with mass (p = 1), growth
would either run away or shut off entirely depending on the sign of
a-b. The fact that animals have a finite asymptotic size at all
requires a sublinear intake term, and the geometric argument for
p = 2/3 is the simplest one available. A more careful biologist
might invoke 3/4 from fractal vascular networks, or species-specific
exponents reflecting body shape. On this dataset the data does not
discriminate between 0.67 and 0.75 at any useful significance.

The other quiet point: a 90-year-old three-parameter equation with
one mechanistic exponent and two empirical constants outperforms a
generic logistic fit by 45 AICc units on a tiny dataset. Models with
mechanism in them tend to fit better than models without, even when
the mechanism is wrong in the details. That seems worth remembering
when reaching for a logistic curve to describe anything
biology-flavoured.

## Loose ends

The cod table is hand-encoded from a published growth pattern, not
from raw otolith reads. I'd want to redo this with the actual ICES
otolith database to see whether per-fish variance changes the AIC
ranking. The free-p fit on length^3 substitutes for real mass
measurements, which is a crude proxy when cod condition factor
varies seasonally. With another week I would pull in published
length-weight regressions for cod and refit p on actual W(t), then
do the same exercise on a non-fish species (say, a mammal with
indeterminate growth) to see whether the 2/3 number holds up
outside the marine context Bertalanffy originally had in mind.
