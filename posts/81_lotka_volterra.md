# Lotka and Volterra's 1925 cycles match small-amplitude theory to 1 part in 10,000, and roughly fit lynx-hare pelts

*Part 81 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Alfred Lotka wrote down a pair of coupled differential equations in
1925, in a book called *Elements of Physical Biology*. Vito Volterra
arrived at the same equations a year later, prompted by his
son-in-law Umberto D'Ancona, who had noticed that the share of
predatory fish in Adriatic catches went up during World War I when
fishing was suspended. The equations were:

    dx/dt = alpha * x - beta * x * y
    dy/dt = delta * x * y - gamma * y

x is the prey, y is the predator. Alpha is how fast the prey would
breed if undisturbed. Gamma is how fast predators would die without
food. Beta and delta govern the meeting and conversion of biomass at
the interface. The system has a non-trivial equilibrium at (gamma /
delta, alpha / beta), but it never settles there. Solutions cycle
forever, in closed orbits.

The textbook claim is famous, and I wanted to see it on a screen
rather than on a page. Three checks: do the orbits actually close?
Does the small-amplitude period come out to 2*pi / sqrt(alpha * gamma)?
And what happens if you fit the four parameters to the Hudson Bay
Company's lynx and snowshoe-hare pelt records, the textbook
illustration that almost every ecology course shows?

## What I tried

I wrote a 4th-order Runge-Kutta integrator from scratch, not because
SciPy doesn't have one but because the system is small enough that
the explicit version is shorter than the wrapper call. The textbook
parameters (alpha, beta, gamma, delta) = (1.1, 0.4, 0.4, 0.1) put
the fixed point at (4, 2.75), and five different initial conditions
trace nested loops in the (x, y) plane.

The Lotka-Volterra system has a conserved quantity,

    V(x, y) = delta*x - gamma*ln(x) + beta*y - alpha*ln(y),

that is constant along every trajectory. A good integrator should
leave V invariant up to numerical noise. RK4 isn't symplectic, so V
drifts a little, but the drift is a useful sanity check.

For the period, the linearisation about the fixed point reduces to a
simple harmonic oscillator with angular frequency sqrt(alpha *
gamma). So the small-amplitude period should be 2*pi / sqrt(alpha *
gamma) = 9.4723 time units, regardless of which orbit you start on,
as long as you start near the fixed point.

The third test is the one I wasn't sure about. The Hudson Bay lynx
and hare series, popularised by Charles Elton and tabulated by D.A.
MacLulich in 1937, runs from roughly 1845 to 1935 and is the
standard advertisement for LV in undergraduate biology. I
hand-encoded the 1845-1903 annual hare and lynx pelt counts (59
points each) from the standard tabulation, then fit (alpha, beta,
gamma, delta) and the two initial conditions by minimising
normalised squared error between the LV simulation and the data.
Six free parameters, 118 observations, so the fit is well
constrained in principle. Whether LV can actually capture the cycle
is another question.

## What happened

The phase portrait is the satisfying part.

![Lotka-Volterra phase portrait](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/81_phase_portrait.png)

Five initial conditions, five concentric closed orbits, all looping
counter-clockwise around the fixed point at (4, 2.75). Over 80 time
units the conserved quantity V drifts by a relative 2.4 x 10^-9.
That's RK4 noise, not physics. The orbits close.

The period check came out tight at small amplitude and loose at
large. At relative perturbation 0.05 from the fixed point the
measured period is 9.4735, against a theoretical 9.4723. The
relative error is 1.4 x 10^-4. Push the amplitude harder and the
period grows: at 0.8 relative perturbation, T climbs to 11.2, about
18% above the linear value. Lotka-Volterra is anharmonic. The closer
you sit to the equilibrium, the more it looks like a textbook
oscillator.

Here is the core of the RK4 plus period scan, with the imports left
out.

```python
def lv(state, a, b, g, d):
    x, y = state
    return np.array([a*x - b*x*y, d*x*y - g*y])

def rk4(f, y0, t, args):
    out = np.zeros((len(t), len(y0))); out[0] = y0
    for i in range(len(t)-1):
        h = t[i+1] - t[i]
        k1 = f(out[i], *args)
        k2 = f(out[i] + 0.5*h*k1, *args)
        k3 = f(out[i] + 0.5*h*k2, *args)
        k4 = f(out[i] + h*k3,     *args)
        out[i+1] = out[i] + (h/6.0)*(k1 + 2*k2 + 2*k3 + k4)
    return out
```

Then the lynx-hare fit, which I had been quietly dreading.

![Hudson Bay lynx-hare with LV fit](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/81_lynx_hare.png)

The fitted parameters came out:

| Parameter | Value |
|---|---|
| alpha (hare growth) | 1.61 yr^-1 |
| beta (predation) | 0.054 |
| gamma (lynx death) | 0.98 yr^-1 |
| delta (conversion) | 0.022 |
| Period 2*pi/sqrt(alpha*gamma) | 5.0 yr |

The Pearson correlation between fitted and observed series is 0.41
for the hare and 0.41 for the lynx. That is, the cycle is in phase
roughly half the time and out of phase the rest, with the fitted
trajectory acting like a smoothed sinusoid laid over a much spikier
record. Hare peaks of 150 (1854) and 137 (1865, 1866) get flattened
to peaks of about 80.

The fitted period of 5 years was the surprise, because every
textbook I have read calls this the "10-year cycle". Looking at the
peak-to-peak intervals in the hand-encoded series, though, the
median spacing between hare maxima is 6 years and between lynx
maxima is 5.5 years. The "10-year" label seems to come from the
dominant Fourier mode over the full 90-year MacLulich tabulation,
not from the local cycle in the first half of the record. LV locked
onto the local cycle, which is closer to half that.

So the model is right about the geometry (closed orbits, conserved
V, period set by sqrt(alpha*gamma)) to many decimal places. It is
roughly right about the lynx and hare cycling. It is not right about
amplitude, peak shape, or the way the lynx sometimes appears to
lead the hare in the data, which is impossible in pure LV.

## What it may mean

Two things, both hedged.

The first is that LV is what mathematicians call a normal form. It
is the simplest set of equations that produces persistent
oscillation in a closed two-species loop. Any predator-prey system
that is sufficiently slow-varying near a stable cycle should look
like LV to leading order, in the same way that any oscillator looks
like a harmonic one near a minimum of its potential. So getting r =
0.41 against real ecology with four free parameters is, on this
sample, neither a triumph nor an embarrassment. It is what you would
expect from a leading-order model fit to a system with omitted
variables.

The second is that the canonical lynx-hare illustration may have
been doing more pedagogical work than empirical work. The original
Volterra paper was about Adriatic fish; the lynx-hare overlay came
later. Looking carefully at the MacLulich numbers, the cycle is
real, the LV model captures its phase imperfectly, and the real
biology probably involves a third trophic level (hare-vegetation)
and a strong climate signal that pure two-species LV cannot encode.
The 10-year cycle has been linked, in various papers, to the
roughly 11-year sunspot cycle, to snow-depth dynamics, and to
multi-decade vegetation regimes. None of those are inside LV.

That doesn't make LV wrong. It makes it the right starting place.

## Loose ends

The hand-encoded series here is one transcription of MacLulich
(1937); other published versions disagree at the level of a few
pelts per year. The fit is non-convex and the differential-evolution
search with seed 0, polished by Nelder-Mead, may have missed a
slightly better minimum. The natural next steps would be a Holling
type-II functional response (which adds a saturation term beta*x*y
/ (1 + h*x)), a logistic prey growth alpha*x*(1 - x/K), and a
proper time-series treatment with observation noise rather than
least-squares on raw counts. You can replicate everything in this
post in about a minute on a laptop.
