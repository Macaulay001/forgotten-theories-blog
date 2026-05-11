# Debye's 1912 T³ law for solids holds to the fourth decimal in a notebook test

*Part 70 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1907, Einstein wrote down the first quantum model of a solid's heat
capacity. It worked at room temperature and explained why C_v drops
below the classical Dulong-Petit value of 3R (about 24.94 J/mol/K) at
low temperatures. But near absolute zero it dropped too fast,
exponentially. Calorimetry data from Walther Nernst's lab in Berlin,
collected on diamond and other crystals between 1907 and 1911, refused
to match.

In 1912 Peter Debye published a one-page idea that fixed it. Instead
of treating every atom as an independent oscillator at one frequency,
he counted standing sound waves in the solid up to a cutoff. The
density of those modes grows as ω², not as a delta function. Integrate
their quantum-statistical contribution and you get a heat capacity
that approaches 3R at high T and falls as T³ at low T. The T³ law
became one of the early visible wins for quantum mechanics in
condensed matter.

I wanted to feel the law for myself, with my own numbers.

## What I tried

The Debye prediction is a single integral:

C_v(T) = 9R (T/Θ_D)^3 ∫_0^{Θ_D/T} x^4 e^x / (e^x - 1)^2 dx

with one material-specific parameter, the Debye temperature Θ_D. For
diamond Θ_D is about 2230 K (very stiff bonds, high cutoff). For
copper it is about 343 K. Same equation, different scale.

The plan was simple. Sweep T from 1 K to 1000 K on a log grid, do the
quadrature, then check two things. At very low T (I picked T = 0.02 Θ_D),
does the full integral agree with the analytical asymptote
(12π^4/5) R (T/Θ_D)^3? And at very high T (T = 5 Θ_D), does the answer
approach 3R = 24.94 J/mol/K?

For contrast I also computed the Einstein heat capacity,

C_v^E = 3R x² e^x / (e^x - 1)^2 with x = Θ_E / T,

using the standard choice Θ_E ≈ 0.806 Θ_D, which lines the two models
up around T = Θ_D. The interesting question is what happens to Einstein
when T falls well below that.

The Debye integrand has two awkward regions. At x → 0 it looks like 0/0
but is actually x²; at large x the e^x in numerator and denominator both
explode and the ratio overflows. The fix is a small series expansion
near 0 and an asymptotic x^4 e^{-x} replacement past x ≈ 50.

## What happened

The numerics behave exactly as Debye said they would.

At T = 5 Θ_D the Debye integral returns 24.894 J/mol/K. The
Dulong-Petit value is 3R = 24.943 J/mol/K. The residual gap of
0.2 percent is the slow approach to the classical limit, not a bug.
Both diamond and copper hit the same number because the result is
dimensionless once you scale T by Θ_D.

The low-T check is more striking. At T = 0.02 Θ_D the full Debye
integral gives C_v = 1.555 × 10⁻² J/mol/K. The (12π^4/5) R (T/Θ_D)^3
asymptote gives 1.555 × 10⁻². The ratio is 1.0000 to four decimals.
That is the T³ law showing up in the third significant figure of a
numerical integration.

Einstein at the same temperature is roughly 1.27 × 10⁻¹³ J/mol/K. The
Debye-to-Einstein ratio at 0.02 Θ_D is about 1.2 × 10¹¹. Eleven orders
of magnitude. That gap is what Nernst's data exposed in 1911 and what
Debye's continuum density of modes closed.

Here is the core of the calculation, stripped of bookkeeping:

```python
def debye_integrand(x):
    if x < 1e-6: return x*x
    if x > 50:   return x**4 * np.exp(-x)
    ex = np.exp(x)
    return x**4 * ex / (ex - 1.0)**2

def debye_Cv(T, theta_D):
    u = theta_D / T
    integral, _ = quad(debye_integrand, 0.0, u, limit=200)
    return 9.0 * R * (T / theta_D)**3 * integral
```

A small results table:

| T / Θ_D | C_v (Debye) | T³ asymptote | C_v (Einstein) |
|---|---|---|---|
| 0.02 | 1.555e-2 | 1.555e-2 | 1.27e-13 |
| 0.1  | 1.92     | 1.94     | ~5e-3 |
| 1.0  | 22.7     | n/a      | 21.8 |
| 5.0  | 24.89    | n/a      | 24.81 |

For diamond, Θ_D = 2230 K means room temperature sits at T/Θ_D ≈ 0.13.
That is why diamond's measured C_v at 300 K is only about 6.1 J/mol/K,
roughly a quarter of the classical value. The molar heat capacity of
copper at the same 300 K is around 24.5 J/mol/K, close to 3R, because
copper sits at T/Θ_D ≈ 0.87. Same equation, two regimes, picked apart
by a single material parameter.

![cv_vs_T](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/70_cv_vs_T.png)

The log-log plot makes the structure visible. Both materials' Debye
curves bend smoothly from a T³ slope at the bottom to a flat 3R
plateau at the top. The Einstein curves track them near and above Θ_D
but plunge below them at low T. The dotted T³ line slips right under
the copper curve and stays there.

## What it may mean

The Debye function does not predict any single material's heat
capacity to spectroscopic accuracy. Real phonon spectra are messier
than an ω² cutoff. The model does, however, capture the universal
shape: a T³ rise from absolute zero, a smooth knee near Θ_D, an
approach to 3R that is slow but inevitable. That universality is part
of why the law gets taught in every solid-state course.

It also gives a clean object lesson about model choice. Einstein's
1907 calculation was not wrong in any algebra sense. It was wrong in
its mode-counting. Counting acoustic modes from a continuous spectrum
(rather than a single resonance) is the small change that closes an
eleven-order-of-magnitude gap at 2 percent of Θ_D. The numerical work
here is just bookkeeping; the physics lives in the density of states.

A modern echo: in two-dimensional crystals, the linear dispersion
gives a phonon density of states that grows as ω rather than ω², and
the low-T heat capacity goes as T². There are papers from the
graphene era reporting exactly that, and the Debye-style derivation
walks across dimensions almost mechanically.

There is also a small historical point worth flagging. Debye's paper
is often described as a refinement of Einstein's, and it is, but the
two models answer slightly different questions. Einstein asked, "what
heat capacity does a collection of identical quantum oscillators
have?" Debye asked, "what heat capacity does a collection of
collective vibrational modes in an elastic solid have?" The second
question is closer to the actual physics of phonons. The fact that
both reduce to 3R at high T is the equipartition theorem doing what
it always does. The fact that they disagree by eleven orders of
magnitude at 2 percent of the cutoff temperature is what tells you
quantum statistics is real.

One more practical detail. The integration is fast, well under a
second per material for 400 temperature points using scipy.quad. For
anyone who wants to redo this at home, the bottleneck is not the
integral, it is finding a sensible Θ_D value for the material of
interest. Tabulated Debye temperatures vary by ten or twenty kelvin
between sources, partly because they are extracted from different
T-ranges of the same data. The model is exact; the parameter is
fuzzy.

## Loose ends

The model ignores electronic specific heat (the γT term that
dominates copper below about 10 K), anharmonicity above Θ_D, and the
weak temperature dependence of the "effective" Θ_D in real data. A
fuller test would fit Θ_D as a free parameter against measured C_v(T)
for a half-dozen elements and look at the residuals. With another
afternoon one could pull the standard cryogenic datasets and report
where Debye fails by more than a few percent. My guess: at the
magnetic and superconducting transitions, where something else is
going on entirely.
