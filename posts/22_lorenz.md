# I rounded Lorenz's initial condition by 1 part in 10^8. The forecast died at t=23.

*Part 22 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In December 1961, Edward Lorenz left a convection simulation running on
his Royal McBee LGP-30 and went for coffee. When he came back, he
restarted the run from a printed midpoint, typing 0.506 instead of the
machine's internal 0.506127. The two trajectories tracked for a few
simulated weeks and then split completely. He thought the vacuum tubes
had failed. They had not. The 1963 paper that followed, "Deterministic
Nonperiodic Flow," is now one of the most cited works in atmospheric
science, though it sat in obscurity for nearly a decade because it ran
in a meteorology journal that physicists rarely read.

The claim is precise enough to test in an afternoon: a perturbation
of one part in 10^4 in the initial state grows exponentially until the
forecast is useless. I wanted to know how that growth rate compares to
what the textbooks now quote (a Lyapunov exponent of 0.9056), and how
much extra forecast horizon you actually buy by measuring more
precisely.

## What I tried

The Lorenz '63 system is three coupled ODEs with parameters sigma=10,
rho=28, beta=8/3. Nothing exotic. I wrote a hand-rolled RK4 integrator
with dt=0.001, spun up a trajectory for 10 time units to settle onto
the strange attractor, then forked it. State A continued unchanged.
State B had its x-component shifted by delta_0, a tiny number ranging
from 1e-12 up to 1e-3 across runs.

The core measurement is the Euclidean separation d(t) between A and B.
On a log plot, d(t) should rise as a straight line with slope equal to
the largest Lyapunov exponent, until d saturates near the attractor
diameter (about 30 across in x). The slope of that line, and the time
at which saturation occurs, are the two numbers I cared about.

For each delta_0 I fit log(d) versus t over a window where d is large
enough to escape numerical noise but small enough to still be in the
exponential regime. I also recorded t_sat, the first time d crossed
10. If the standard chaos picture is right, t_sat should scale as
(1/lambda)·ln(D/delta_0) with D around 10.

Nothing about this is novel research. It is a calibration exercise:
do the textbook numbers actually fall out of a clean reproduction, or
are they slightly off in a way that matters?

## What happened

For a 1e-8 perturbation, separation reached d>10 at t=22.76 time
units. The fitted growth rate was 0.877 per time unit, which is
within 3% of the published 0.9056.

The cleaner numbers came from intermediate perturbations:

| delta_0 | fitted lambda | t_sat |
|---------|---------------|-------|
| 1e-12   | 0.901         | 31.9  |
| 1e-9    | 0.895         | 25.1  |
| 1e-8    | 0.877         | 22.8  |
| 1e-6    | 0.859         | 17.5  |
| 1e-4    | 0.487*        | 11.5  |
| 1e-3    | 0.568*        | 8.6   |

The starred rows have a short exponential window because the
trajectory saturates quickly, so the fit underestimates lambda. If you
restrict to the four small-delta rows, the slope clusters around
0.88-0.90.

The scaling law also held. Predicting t_sat from
(1/0.9056)·ln(10/delta_0) gives 32.9, 25.4, 22.9, 17.8 for the
small-delta rows, compared to measured 31.9, 25.1, 22.8, 17.5. Within
one time unit across four orders of magnitude in delta_0.

Here is the part of the simulation that actually matters:

```python
def divergence(delta0, T=40.0, dt=0.001):
    s = spun_up_state()
    a, b = s.copy(), s.copy()
    b[0] += delta0
    n = int(T/dt)
    d = np.empty(n+1)
    d[0] = abs(delta0)
    for i in range(n):
        a = rk4_step(a, dt)
        b = rk4_step(b, dt)
        d[i+1] = np.linalg.norm(a - b)
    return d
```

Two trajectories, one perturbed by delta_0, the same map applied to
both. The exponential growth is not a property of the integrator (you
get the same slope with scipy's LSODA, with explicit Euler at small
dt, or by hand with leapfrog). It is a property of the attractor's
local divergence.

What surprised me was how few orders of magnitude in delta_0 buy you
extra predictability. Going from 1e-3 to 1e-12 (nine decimal places of
better measurement) extends the useful forecast from t≈9 to t≈32. That
is a factor of 3.6 in horizon for a factor of 10^9 in instrument
precision. In atmospheric units, where one Lorenz time unit is roughly
a day on the synoptic scale, this maps to extending a 9-day forecast
to a 32-day forecast at the cost of instruments nine orders of
magnitude better. Nobody is buying that, which seems to be why
operational weather forecasting still tops out near two weeks despite
fifty years of sensor improvements.

One detail worth flagging: at delta_0=1e-12 the fit slightly
overshoots (0.901 vs 0.876 at 1e-8). That row is brushing against
RK4's own truncation error at dt=0.001, which I estimate at around
1e-12 per step. So the very smallest perturbation may be measuring
numerical noise as much as physical chaos. With double precision and
this step size, you cannot probe much below that without switching to
extended arithmetic.

## What it may mean

The butterfly effect, as Lorenz described it, is not metaphor. In the
toy model it is a quantitative law: separation grows as
exp(0.9·t) until d saturates. If real atmospheric dynamics behaves
even loosely like Lorenz '63 on its slowest unstable mode, then the
ceiling on forecast horizon is logarithmic in observation precision.
You can extrapolate the rough number: every factor-of-10 reduction in
initial-condition error buys roughly ln(10)/0.9 ≈ 2.5 extra time
units. No more.

This is consistent with operational results from ECMWF and NOAA, which
have reported steady but slowing gains in forecast skill over decades
as both models and sensors improved. The bottleneck is not compute;
it is the exponential amplification of whatever uncertainty the
initial condition carries.

The result also says something more philosophical about determinism.
The Lorenz system is fully deterministic. No noise term, no quantum
indeterminacy, no chaos in the loose sense. And yet a perturbation in
the twelfth decimal place dominates the trajectory by t=32. The
information content of "where the system goes next" exceeds the
information you can store in your initial condition, regardless of how
precise that condition is, because new bits flow up from below the
measurement floor at a rate fixed by lambda.

There is also a quieter point about scientific publishing. Lorenz's
1963 paper sat almost unread for about a decade. It appeared in the
Journal of Atmospheric Sciences, which physicists and mathematicians
of the period generally did not scan. James Gleick's 1987 popular
book did more to spread the result among non-meteorologists than the
paper itself. If a finding this consequential can hide in a
discipline-specific journal for ten years, it seems reasonable to
suspect that other quantitative claims of similar weight are sitting
in equally obscure places right now, waiting for someone to type them
into an integrator.

## Loose ends

A few things I would do with another week. First, check the
correlation dimension of the attractor by box-counting on the saved
trajectory (the published value is roughly 2.06). Second, run the
same protocol with stochastic forcing added, the way modern ensemble
forecasts do, and see how the spread compares to the deterministic
divergence. Third, port this to the higher-dimensional Lorenz '96
toy, where the predictability horizon behaves differently because
the spectrum of Lyapunov exponents is richer. Fourth, redo the
smallest-delta runs in quad precision to see whether the apparent
slope of 0.90 at delta_0=1e-12 is real or a numerical artifact. None
of these would change the basic finding: Lorenz was right, and his
1963 number survives reproduction at 2026 precision to within about
3%.
