# I refit Michelson and Morley's 1887 fringe table to the stationary-aether prediction. The aether-wind sinusoid sits 114 standard deviations away from what they measured.

*Part 103 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1864 Maxwell wrote the equations for light and stated, plainly, that they required a medium. The medium had a name by then: the luminiferous aether, Fresnel's stationary substance that filled all of space and carried electromagnetic waves the way air carries sound. The aether had a measurable rest frame. Earth, moving at 30 km/s around the Sun, had to be sweeping through it. The wind should be visible to anyone who could measure a length to one part in 10^8.

Michelson and Morley tried in 1887. They built an interferometer with arms folded by multiple mirrors to an effective 11 m, floated the whole thing on mercury so it could be rotated without strain, and watched the fringes through a telescope. The aether predicted a sinusoidal fringe shift with amplitude about 0.4 of a fringe as the apparatus turned through 360 degrees. They reported a shift of at most a few hundredths of a fringe, called the result a null, and went back to other projects. By 1920 the aether was gone from physics.

I have read this story many times and have never seen the actual 1887 table reanalyzed with modern statistics. So I encoded it by hand from their published Table III and refit it.

## What I tried

The aether-wind prediction is straightforward. A Michelson interferometer with arm length L, illuminated at wavelength lambda, oriented at azimuth theta relative to Earth's motion through a stationary medium, should show a fringe shift

    Delta N(theta) = (2 L / lambda) * (v^2 / c^2) * cos(2*(theta - theta0))

with theta0 set by the direction of motion. The factor of 2 in `2*theta` is geometric: rotating 180 degrees brings the apparatus back to its original axis. For Michelson and Morley's setup with sodium D-line lambda = 590 nm, L = 11 m, and Earth's orbital v = 30 km/s, that is

    (2 * 11 / 5.9e-7) * (1e4 / 3e8)^2 * (3e4 / 1e4)^2
      ~ 3.7e7 * 1.0e-8
      ~ 0.37 fringes.

So if their data scatter is less than something like 0.1 fringes, the null is robust. If the scatter is 0.5, you can't tell either way.

Their Table III gives the combined fringe deflections at 16 azimuths, in two sittings (noon and evening). I typed in both rows, sixteen numbers each. The full series fits in two NumPy arrays. The combined fringe shifts range from -0.015 to +0.023 fringes, all comfortably below the predicted 0.37.

For the test I fit a free-amplitude sinusoid of the form `a*cos(2*theta) + b*sin(2*theta) + c` by least squares, bootstrapped over the 16 azimuths to get an uncertainty on the amplitude, and computed the per-point residual SD. Two noise scales fall out: the bootstrap SD on the fitted amplitude (0.0018 fringes) and the white-noise estimator SD given the residual scatter (0.003 fringes). The aether prediction is a fixed number with no error bar; the z-score is just (predicted - observed) / noise.

A separate piece of the script computes the Sagnac fringe shift for the same arm length folded into a square loop. This is the corner that survived 1887. Rotation produces a fringe shift; translation through a stationary medium does not. The formula is

    Delta N = 4 A Omega / (lambda c)

for a loop of area A rotating at Omega. With A = 30.25 m^2 and Earth's daily rotation, that gives 5e-5 fringes, an effect Michelson would not have seen even if his apparatus had been a loop. With Omega = 1 revolution per second it gives 4.3 fringes, larger than the aether prediction it was meant to disprove. Modern ring-laser gyroscopes in commercial aircraft are built on exactly this geometry.

## What happened

The fit to the combined Michelson-Morley series gives an amplitude of 0.0043 fringes, against the predicted 0.373. The estimator SD under white noise is 0.0032 fringes, so

    z = (0.373 - 0.004) / 0.0032 ~ 114.

A small table of the relevant numbers:

| quantity | value |
|----------|-------|
| predicted amplitude | 0.373 fringes |
| observed amplitude (combined fit) | 0.004 fringes |
| per-point residual SD | 0.009 fringes |
| estimator SD on amplitude | 0.003 fringes |
| z-score, prediction vs observed | 114 |
| ratio observed / predicted | 1 / 80 |

The core of the fit is a few lines:

```python
def fit_sinusoid(theta_deg, y):
    th = np.deg2rad(2 * theta_deg)
    X = np.column_stack([np.cos(th), np.sin(th), np.ones_like(th)])
    coef, *_ = np.linalg.lstsq(X, y, rcond=None)
    a, b, c = coef
    amp = math.hypot(a, b)
    phase = 0.5 * math.degrees(math.atan2(b, a))
    return amp, phase, c
```

The z-score of 114 is much higher than the 20 sigma figure I was expecting before running the fit. The reason is that the 16 azimuths pin a sinusoid very tightly. Even with the per-point scatter at 0.009 fringes, the amplitude of a free-phase cos(2*theta) is constrained to a few thousandths. Michelson and Morley were not exaggerating when they said the wind was not there. They had the precision; the wind genuinely was not there.

Two caveats temper the z-score down a notch. First, the residuals are not white; thermal drift over the rotation cycle introduces correlated structure that Shankland and coauthors picked apart in 1955. If the effective noise scale is two or three times larger, the z-score halves but stays well above 30. Second, I encoded the table from the published summary, not the original notebooks (digitized by Case Western, more azimuths). Either correction moves the result by less than a factor of 3, and the prediction is off by a factor of 80.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/103_fringe_shift_comparison.png)

The plot makes it visual. The crimson curve is the aether prediction. The data points sit on the zero line at the resolution of the plot. The black sinusoid is the best-fit free-amplitude oscillation, with the right phase but the wrong size by almost two orders of magnitude.

The right panel is the Sagnac check. The blue line shows fringe shift versus rotation rate for a closed loop of the same arm length. The aether prediction (crimson dashed) and the 1887 observed (black dotted) sit at fixed values that translation alone cannot reach. But once you rotate the loop, even at Earth's daily spin rate, you get a small but real signal. By 1 rev/s the rotational shift exceeds the aether amplitude that the 1887 data could not find.

This is the asymmetry that often gets lost in the standard "Michelson-Morley killed the aether" story. They did not rule out a preferred frame in general. They ruled out a translational aether wind, the version Fresnel and Maxwell had specifically committed to. Rotation produces fringe shifts that look, mathematically, like the kind of thing the aether was supposed to do, and modern navigation hardware uses them every day.

## What it may mean

A few things follow that I would not have written before doing the arithmetic.

The Michelson-Morley result was not "marginal" or "barely null". Treating their 16-azimuth table as a fresh dataset, the aether-wind prediction is rejected at a level that no plausible noise model can rescue. The Lorentz-FitzGerald contraction, which shrinks the parallel arm by exactly `sqrt(1 - v^2/c^2)` to cancel the predicted shift, had to do real work; you cannot wave it away as a small correction. Einstein 1905 reabsorbed the same factor as kinematic, not material, and the aether stopped being necessary.

The Sagnac side is what I find most interesting. The reason no high-school physics class mentions the aether anymore is that nothing depends on it. The reason every commercial pilot does depend on something aether-shaped is that rotation breaks the symmetry that translation cannot. Both facts come out of the same interferometer geometry, set up two different ways.

On this single re-analysis I would not claim the original 1887 data secretly held a small but real signal at the 0.004-fringe level. A few thousandths is exactly the regime where bench drift dominates, which is the point Shankland made seventy years later.

## Loose ends

The honest next step is to pull the original Michelson-Morley notebook scans from the Case Western archive, encode all 36 azimuth runs rather than the 16 in the published summary, and run a proper colored-noise analysis. That would tighten or widen the z-score by a factor of two or three. It would not put the aether back. The interesting modern question, which I did not try here, is whether any Sagnac-style experiment has set a competitive bound on Lorentz-violation parameters in the post-relativity sense; that lives in the Standard Model Extension literature and would take more than an afternoon.
