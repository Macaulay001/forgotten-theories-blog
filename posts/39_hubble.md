# I refit Hubble's 1929 galaxy table. The slope is six times too steep, and the line is still real.

*Part 39 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1929 Edwin Hubble published three pages in PNAS with a small table and a
scatter plot. Twenty-four galaxies, two columns of numbers, one straight
line drawn through them. He claimed the recession velocity of a galaxy
grows in proportion to its distance, v = H_0 d, and read off a slope of
roughly 500 km/s per megaparsec. That slope implied a universe about
two billion years old, which was younger than the rocks under his feet.
The shape of the relation was right and the number was famously wrong,
and I wanted to see what the original data actually contains when you
hand it to numpy in 2026.

So I typed Hubble's Table 1 into a Python list. Name, distance in
megaparsecs, radial velocity in km/s. The smallest entries are the
Magellanic Clouds at 0.03 Mpc. The largest are four members of the
Virgo cluster, all listed at 2 Mpc, with velocities between 500 and
1090 km/s. A few nearby galaxies (M31, M32, M33) come in with negative
velocities, because they are bound to us and moving inward. Hubble
knew this and kept them in.

## What I tried

The plan was small and direct. Fit two versions of the law and see how
much they disagree. First a forced-through-origin fit, v = H d, which is
the form Hubble preferred. Second a regular least-squares fit with an
intercept, which lets the data argue for a non-zero offset if it wants
one. Then compare both numbers to the modern range, 67 km/s/Mpc from
Planck CMB analysis to 73 km/s/Mpc from the SH0ES distance ladder.

I also added a second panel. The 1929 sample reaches z roughly 0.004
at most. That is deep in the linear regime, and any cosmology that
expands at all will look like a straight line here. To show where the
law stops being a straight line I generated a synthetic Type Ia
supernova sample out to z = 1.2, drawing luminosity distances from a
flat Lambda-CDM integral with Omega_m = 0.315 and H_0 = 70. Adding
7% scatter to each distance gives something that looks roughly like a
miniature Pantheon plot. The point is not to rediscover dark energy,
the point is to show that a strict v = H_0 d line cannot fit both
ends of the redshift range at once.

I did not use an LLM for any of the numerical work. The analysis is
24 hand-typed rows from the original paper plus a 60-point mock.

## What happened

The 1929 data, fit through the origin, gives H_0 = 424 km/s/Mpc.
With a free intercept the slope rises slightly to 454 km/s/Mpc and
the intercept lands at -41 km/s. R^2 is 0.62, which is not amazing
but is clearly non-zero. The line is real in his data.

```python
# Hubble's 24 nebulae, forced through origin
H_noint = np.sum(v * d) / np.sum(d * d)        # 423.9 km/s/Mpc
slope, intercept = np.polyfit(d, v, 1)         # 454.2, -40.8
r2 = 1 - ((v - (slope*d + intercept))**2).sum() \
        / ((v - v.mean())**2).sum()            # 0.624
```

The headline number is 6.1x too large. Modern best estimates put H_0
near 70 km/s/Mpc. Where did the factor of six come from? Almost all of
it is in the distances, not the velocities. Hubble's velocities were
spectroscopic Doppler shifts, and those have held up well. The
distances came from Cepheid variable stars calibrated against a
period-luminosity relation that Henrietta Leavitt had built from the
Small Magellanic Cloud in 1912. Walter Baade showed in 1952 that there
are two populations of Cepheids, with different intrinsic brightnesses,
and Hubble had been comparing apples to oranges. Correcting that
doubled the distance scale immediately. Allan Sandage spent the next
two decades pulling the scale further out, and by 1960 H_0 was
estimated near 100. It did not settle near 70 until the Hubble Key
Project ran in the late 1990s.

The breakdown is most obvious at the far end of his table.

| Subset | Mean d (Mpc) | Mean v (km/s) | Predicted at H_0 = 70 |
|---|---|---|---|
| Virgo-region galaxies, d >= 1.5 | 1.94 | 840 | 136 |
| Hubble's own fit at d = 1.94 | 1.94 | 840 | (matches by construction) |

So the law's slope was calibrated against galaxies whose distances
were under-counted by a factor of six. Those Virgo galaxies are
actually about 16 Mpc away, not 2 Mpc. Hubble's velocities are
roughly fine, his distances were short, and the ratio that comes out
is therefore high.

The synthetic supernova panel shows the second failure mode of the
law. Below z = 0.1 the points fall on a clean line. Above z = 0.3 a
v = 70 d line starts under-predicting the velocity needed to match
the luminosity distance, and by z = 1 the deviation is large. This
is the regime where the cosmological constant matters, and it is also
the regime that defeats any interpretation of Hubble's law as a
literal special-relativistic Doppler effect. The "velocities" at
those distances are coordinate quantities in an expanding metric, not
local motions through space.

![Hubble's 1929 fit on the left, mock Type Ia SNe Lambda-CDM on the right](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/39_hubble_fit.png)

## What it may mean

The qualitative claim that v scales with d in the local universe is
about as well-supported as any empirical law gets. The 1929 data shows
it weakly, the 1990s Cepheid calibration shows it strongly, and the
2020s gravitational-wave standard sirens show it independently. The
linear relationship survives every recalibration. The numerical value
of H_0 has been the moving target.

The interesting thing about Hubble being off by 6x is that nobody
threw out the law. They tightened the rungs of the distance ladder.
Bottom rung is parallax inside the Milky Way. Next is Cepheids in
galaxies close enough to resolve. Next is Type Ia supernovae in
those Cepheid-calibrated galaxies, which lets you go to z > 1. Each
rung inherits the calibration errors of the rungs below it, which is
why the current 5-sigma tension between Planck's 67.4 and SH0ES's
73.0 may yet turn out to be a systematic on rung two or three rather
than new physics. JWST's first batch of Cepheid recalibrations in
2024 nudged SH0ES very slightly toward Planck but did not close the
gap.

If the gap is real, it points at something missing in the standard
Lambda-CDM model between recombination and now, possibly early dark
energy, possibly a non-trivial neutrino sector. That is the live
frontier the 1929 paper still opens.

## Loose ends

The high-z panel here is a mock, not actual Pantheon+ data. Plotting
real SN distance moduli with their published errors would tighten the
visual argument and probably take two more hours. I also assumed
Hubble's distances were correct as published when computing his
slope, which is the right thing to do for replicating the 1929 fit
but is exactly the wrong thing to do for interpreting what those
galaxies actually are. The Virgo cluster in his table is the same
Virgo cluster at 16 Mpc, with the same velocities. The slope is fine.
The metersticks were short.
