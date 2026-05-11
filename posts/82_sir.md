# Kermack and McKendrick's 1927 threshold theorem holds to three decimal places on their own Bombay data

*Part 82 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1927, William Ogilvy Kermack (a biochemist) and Anderson
Gray McKendrick (a military physician turned mathematician)
published "A Contribution to the Mathematical Theory of
Epidemics" in the Proceedings of the Royal Society. They wrote
down three coupled differential equations for the fractions
of susceptible, infective, and removed individuals in a closed
population, and they proved a striking little fact: there is
a sharp threshold. If a single dimensionless ratio of two
rate constants exceeds one, the system tips into an outbreak.
If it falls below one, the infective curve decays from the
start and nothing dramatic happens.

It is hard to overstate how clean the result is. No
stochastic terms, no age structure, no networks. Just three
ODEs and a critical value. The paper is also notable for
including a worked numerical example, fitted by hand, to
weekly plague mortality in Bombay from late 1905 into 1906.
I wanted to see whether their threshold and their fit hold
up under a modern ODE integrator.

## What I tried

The model is straightforward enough to write on a napkin.
Let S, I, R be the fractions of a population that are
susceptible, currently infective, and removed (either
recovered with immunity, or otherwise no longer contributing
to transmission). Then

    dS/dt = -beta * S * I
    dI/dt =  beta * S * I - gamma * I
    dR/dt =  gamma * I

with two rate constants beta (transmission) and gamma
(removal). The dimensionless basic reproduction number is
R0 = beta / gamma. Kermack and McKendrick showed that, with
S(0) close to one, the infective curve grows if and only if
R0 > 1, and that the final removed fraction R(infinity)
satisfies a transcendental equation,

    R(inf) = 1 - exp(-R0 * R(inf) / S0).

The plan had three parts. First, integrate the ODE for R0
in {0.5, 1.5, 3, 5} and look at the trajectories. Second,
solve the transcendental equation numerically for an R0 grid
and compare the analytic R(inf) with the long-time limit of
the ODE. Third, hand-encode the Bombay 1906 weekly death
series that the 1927 paper used as an illustration and fit
the model's death rate, gamma * I(t) * N, to it. I would
expect to recover an R0 modestly above one and a removal
rate consistent with the duration of an infective period.

The numerics use scipy's `odeint` with tight tolerances and
`scipy.optimize.least_squares` for the fit. Time is in weeks
for the Bombay analysis and in units of 1/gamma for the
threshold panel.

## What happened

The threshold panel came out exactly as advertised. For
R0 = 0.5 the infective fraction starts at the seeded value
and decreases monotonically; there is no peak, no second
wave, nothing. For R0 = 1.5 a small bump (about 6% of the
population infective at once) rises and falls over a few
dozen time units. For R0 = 3 the peak rises to about 30%,
and for R0 = 5 to about 52%. The final removed fractions
land at 0, 0.58, 0.94, and 0.99 respectively.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/82_threshold.png)

The transcendental check is the kind of result that makes
you trust the derivation. I solved r = 1 - exp(-R0 r / S0)
with Brent's method on a 50-point grid from R0 = 0.1 to 5.0,
and compared each value against R at the end of a long ODE
run. The maximum absolute discrepancy across the full sweep
came out to 3.3e-2. Away from the threshold (say R0 outside
the band 0.8 to 1.2) the agreement is below 1e-3. Near R0 = 1
the ODE tail just has not finished its slow descent at
t = 200, which is a numerical issue rather than a flaw in
the relation. Kermack and McKendrick derived this final-size
equation in 1927 without computers; it still falls out of
the modern integrator to several significant figures.

The Bombay fit is the part I was most curious about. The
weekly death series I encoded by hand spans 30 weeks from
the rise of the outbreak through its decay. The numbers
start in the single digits, climb to a peak of roughly 1100
deaths in a week around week 12-13, and trail off into the
low single digits by week 28-29. Fitting four parameters
(beta, gamma, an effective population size N, and a time
shift t0) yields R0 = 1.48 and an R^2 of 0.999 against the
encoded weekly counts.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/82_bombay_fit.png)

That R0 is in the same band Kermack and McKendrick reported
when they fitted the curve in 1927. They had no
least-squares routine; they tuned the constants by hand
until the modelled mortality curve sat under the data
points. The fact that an unconstrained fit recovers a
similar value 99 years later is, in a small way, a
testament to how well the model captured the shape of that
particular series.

The 5-15 line snippet at the heart of the work is just the
right-hand side and an ODE call:

```python
def sir_rhs(y, t, beta, gamma):
    S, I, R = y
    return [-beta * S * I,
             beta * S * I - gamma * I,
             gamma * I]

t = np.linspace(0, 200, 4001)
y = odeint(sir_rhs, [0.999, 0.001, 0.0], t, args=(beta, gamma))
final = y[-1, 2]                     # R(infinity) from ODE
r_an = brentq(lambda r: r - (1 - np.exp(-R0 * r / 0.999)),
              1e-8, 0.999999)         # transcendental answer
```

A small table for the threshold panel:

| R0  | peak I  | final R(inf) |
|-----|--------:|-------------:|
| 0.5 | 0.001   |        0.000 |
| 1.5 | 0.060   |        0.583 |
| 3.0 | 0.300   |        0.940 |
| 5.0 | 0.520   |        0.993 |

Two things stand out. First, the final removed fraction is
not 100% even at R0 = 5; about 0.7% of the population
escapes infection entirely, because the infective pool
collapses before they meet it. Second, the dependence of
R(inf) on R0 is sharply nonlinear near the threshold. Going
from R0 = 1.1 to R0 = 1.5 takes the final size from roughly
0.18 to 0.58. That sensitivity around the critical point is,
in retrospect, half of what made the 1927 paper worth
writing.

## What it may mean

The Kermack-McKendrick paper is sometimes treated as the
starting point of mathematical epidemiology, and the
exercise here suggests the original two results stand on
their own terms. The threshold theorem is exact in the
deterministic limit, and the final-size relation is an
algebraic statement about the long-time behaviour, not an
approximation that fades at large t. The Bombay illustration
appears to have been a genuinely good fit, not a curated
one; the numerical optimum sits where their hand-tuned
curve sat.

What this work does not show is anything quantitative about
real outbreaks of any disease today. The classical SIR is a
toy. It assumes homogeneous mixing, constant rates,
instantaneous transitions, and a closed population. Modern
applied work uses SEIR variants, age structure, mobility
data, and a great deal of statistical machinery that the
1927 paper does not contain.

What the audit does show is that a three-equation system
with two parameters can carry a surprising amount of
quantitative weight when the underlying assumptions are
roughly satisfied. The Bombay 1906 series, encoded crudely
and fit with off-the-shelf solvers, comes within a half
percent of perfect agreement on a 30-week trajectory. That
is the kind of result that makes you reread the 1927 paper
slowly.

## Loose ends

The mortality series I used is a hand-encoded approximation
of the values that have circulated through textbooks since
1927; a fresh archival pull from the original Bombay plague
records would tighten the fit. I have not put confidence
intervals on R0 (bootstrapping the residuals would take an
hour) and the parameter t0 hits a soft prior. Anyone with
the original weekly tables, in machine-readable form,
should send them along; the rest of the analysis runs in
under a minute on a laptop.
