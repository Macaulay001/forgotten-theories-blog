# I refit Kepler's 1619 harmonic law on the 8 planets and got a slope of 1.4999. The Galilean moons gave Jupiter's mass to 0.3%.

*Part 54 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Johannes Kepler spent most of the 1610s grinding through ratios. He had
Tycho Brahe's planetary tables, a stack of paper, and a conviction that
the spacing of the planets had to encode some musical proportion. In
1619, in book five of *Harmonices Mundi*, he announced what he called
the harmonic law: for the planets, the square of the orbital period is
proportional to the cube of the mean distance from the Sun. In modern
notation, T squared scales like a cubed, so on a log-log plot the
relationship should be a straight line with slope 3/2.

Kepler had only six planets to work with. Uranus and Neptune were
unknown, exoplanets were three and a half centuries away, and the Sun's
mass would not be measurable in any useful sense until Newton showed
how. I wanted to know what the law looks like now, on the cleanest
distance and period numbers we have, and what happens when you point
the same fit at other systems.

## What I tried

The plan is embarrassingly small. Hand-encode the semi-major axes (in
AU) and orbital periods (in years) for the eight Solar planets from
the NASA planetary fact sheet. Fit log10 T against log10 a by least
squares. Read the slope.

Then do it again for a different central body. The four Galilean moons
of Jupiter, the ones Galileo saw in January 1610, have well-measured
orbital radii in thousands of kilometers and periods in days. If
Newton's derivation of Kepler is right, those moons should also lie on
a 3/2 line, but the intercept should be shifted by the mass ratio of
Jupiter to the Sun. The slope tests the inverse-square force law. The
intercept tests the central mass.

Then I added a small bag of exoplanets, mixing hot Jupiters around
Sun-like stars (51 Peg b, HD 209458 b, HD 189733 b), temperate planets
around Sun-like or near-Sun-like stars (Kepler-22 b, Kepler-186 f), and
three planets around M-dwarfs (TRAPPIST-1 e, Proxima Cen b,
GJ 1214 b). The M-dwarf hosts are perhaps 0.08 to 0.12 solar masses.
On a single Kepler fit they should pull the regression off the Solar
line. I wanted to see by how much.

This is not a discovery test. The law has been re-checked thousands of
times. What I wanted was a feel for how clean the law actually is when
you sit down and do it from scratch, and what the residuals look like
when you mix systems with different central masses.

## What happened

The Solar fit came back boring in the best possible way.

```python
# log-log fit of period against semi-major axis
la = np.log10(a)            # AU
lT = np.log10(T)            # years
slope, intercept = np.polyfit(la, lT, 1)
pred = slope * la + intercept
rms = np.sqrt(np.mean((lT - pred) ** 2))
# slope = 1.4999, intercept = -0.0000, log-RMS = 0.0001
```

Slope 1.4999, intercept essentially zero, log-residual RMS of
0.0001 across all eight planets from Mercury at 0.39 AU out to
Neptune at 30.07 AU. The deviation from 3/2 is in the fourth decimal.
The intercept being zero is not a coincidence; it is the statement
that in units of AU and years, the constant 4 pi squared over G times
the Sun's mass equals one.

The Galilean moons gave a slope of 1.5003 when fitted in their own
units (thousands of kilometers and days). Converted to AU and years
the slope is the same, but the intercept jumps to +1.51. That number
is interesting. The relation T^2 = (4 pi^2 / G M) a^3 says that in
AU and years, the prefactor on a^3 is 1 / (M / M_sun). Read the
intercept as 10^(2b) and you get the central mass in solar units. The
Galilean intercept implies a central mass of 9.52 x 10^-4 solar
masses. The accepted mass of Jupiter is 9.5479 x 10^-4 solar masses.
The fit knows Jupiter's mass to about three parts in a thousand,
using nothing but four moons' orbits and a least-squares line.

| System | n | Slope | Implied central mass |
|---|---|---|---|
| Solar planets | 8 | 1.4999 | 1.000 M_sun |
| Galilean moons | 4 | 1.5003 | 9.52 x 10^-4 M_sun |
| Exoplanets (mixed) | 8 | 1.317 | not meaningful |

The exoplanet line is where things got fun. Pooling all eight on a
single fit gave a slope of 1.32 with a log-RMS of 0.19, much messier
than the Solar planets. That looks at first glance like a violation,
but it is not. Each star defines its own Kepler line; the slope on
each line is 3/2, but the intercepts differ because the central
masses differ. Pooling them into one regression flattens the slope,
because the M-dwarf planets sit below the Sun-like planets in the
T-vs-a plane. If you drop the three M-dwarf hosts and refit the
remaining five (all hosted by stars within 30% of one solar mass), the
slope moves back toward 1.5. The Solar law is fine. What the pooled
fit catches is the variation in M.

![log-log T vs a for three Kepler systems](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/54_kepler_loglog.png)

The figure makes the geometry obvious. The Solar planets sit along
the dashed reference line T = a^{3/2}. The Galilean moons sit on a
parallel line offset downward, because at the same a a moon of
Jupiter takes much less time than a planet of the Sun. The exoplanets
straddle the Sun-like line where the host is roughly solar, and drift
toward and below the Jovian line where the host is an M-dwarf, because
an M-dwarf is closer to Jupiter's mass than to the Sun's.

A short note on what the slope is actually telling us. Newton's
derivation says that a central force scaling like 1/r^n produces
closed elliptical orbits only for n = 2 (and n = -1, the harmonic
oscillator). The exponent in Kepler's law is fixed by that 1/r^2.
Empirically refitting log T against log a is therefore a check on the
form of gravity itself. A slope of 1.5 to four decimals across 75
times the range of a is a quiet, century-old constraint on departures
from inverse-square.

## What it may mean

The harmonic law is the cleanest empirical relationship in the
pre-Newtonian record. Slope 1.4999 on the modern numbers, slope
1.5003 on a completely different central body. That kind of agreement
across nine orders of magnitude in mass is rare enough that I think
it is worth pausing over. Kepler did not have the dynamics; he had
ratios. He saw that the geometry alone could be made to fit one
exponent, and he picked the right one. The mechanism waited 68 years
for Newton, but the empirical fact was already locked in.

A practical consequence is that this is how you weigh distant stars.
Given a planet's period and semi-major axis, the intercept of the
Kepler line gives M directly. Most exoplanet catalogues use exactly
this inversion to report host-star masses when other estimates are
unavailable. The 0.3% agreement on Jupiter, from four moons, is
roughly the precision you would expect for a star with a half-dozen
well-timed transiting planets and reasonable distance estimates.

What I cannot rule out is whether the residual at the fourth decimal
of the Solar slope is genuine. With eight planets the standard error
on the slope is dominated by how I encoded the input values, not by
any physics. Mercury's perihelion precession is the famous example of
where Newton-Kepler breaks down, and the deviation there is at the
arc-second level, far below what a slope fit on semi-major axes can
see.

## Loose ends

The exoplanet sample is small and chosen for illustration. A real
test would pull the NASA Exoplanet Archive in bulk, group by host
mass, and check the per-host slopes. I did not do that here; the
point was to see whether the eight Solar planets and four Galilean
moons reproduced Kepler at all. They did, to four decimals. With
another week I would refit roughly a thousand confirmed exoplanets
star by star and look at the distribution of fitted slopes; the
prediction is that almost all of them sit within a percent of 3/2,
and the spread of intercepts maps the stellar mass function.
