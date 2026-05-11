# I refit Snell's 1621 law on 20 noisy air-water angles and recovered n = 1.333 +/- 0.010.

*Part 55 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1621 Willebrord Snell, a Dutch astronomer better known for measuring the meridian by triangulation, worked out a clean relation for how light bends when it crosses from one transparent medium into another. He never published it. The note sat in a manuscript until Christiaan Huygens dug it up decades later. Meanwhile in 1637 Descartes published the same relation in his *Dioptrique*, derived from a mechanical analogy involving tennis balls passing through cloth, with no credit to Snell. A generation after that Pierre de Fermat showed the law also falls out of a single principle: light takes the path that minimizes travel time. Three different derivations, one equation:

    n1 sin(theta1) = n2 sin(theta2)

I had taught this equation. I had drawn the little kink in the ray at the water surface a hundred times. I had never actually fit it to data and I had never numerically minimized Fermat's travel-time integral to watch Snell's law fall out of it. Both take about twenty lines of code, so on a slow afternoon I wrote them.

## What I tried

The plan was three small tests of the same claim.

First, recover n2 from synthetic measurements. I picked the air-to-water interface (n1 = 1.0, n2 = 1.33), generated 20 incidence angles spread evenly from 0 to 80 degrees, used Snell exactly to get the true refracted angle for each, then added Gaussian noise with sigma = 1 degree to each refracted angle. That noise level is roughly what a careful undergraduate gets with a protractor and a laser pointer in a tray of water. With that synthetic dataset I fit n2 by least squares using the linear form sin(theta1) = (n2/n1) sin(theta2). The slope through the origin is n2/n1, so the fit is one line.

Second, look at the critical angle going the other way. From water into air, n2 sin(theta2) = n1 sin(theta1) implies that as theta2 grows, sin(theta1) eventually exceeds 1 and there is no real refracted ray. The handover point is theta_c = arcsin(n1/n2). For water-air this is arcsin(1/1.33) = 48.75 degrees. Above it, light is fully reflected back into the water. This is the principle that keeps light inside optical fibers.

Third, do Fermat directly. Place a source at (-1, 1) above a flat interface at y = 0 and a sink at (+1, -1) inside water. For any candidate crossing point x on the interface, compute the total optical path: euclidean distance in air, plus n2 times euclidean distance in water (since optical path equals geometric path times index). Scan x and find the minimum. From the geometry, read off the incidence and refraction angles. If Fermat and Snell agree, then n1 sin(theta1) = n2 sin(theta2) should hold at the minimum to numerical precision.

Each test is independent of the others. The first checks that the algebraic form survives reasonable measurement noise. The second checks the boundary behavior. The third checks that a least-time variational principle reproduces the same algebra Descartes got from wavefront matching.

## What happened

The fit recovered n2 = 1.333 +/- 0.010. The true value used to generate the data was 1.330, so the estimate sits 0.3 sigma from truth. Residual RMS on the sin(theta1) axis works out to about 1.25 degrees in angle units, which is exactly the noise I put in. Twenty noisy measurements pin the refractive index to better than 1 percent. With sigma = 0.5 deg or with 100 points you would get sub-0.5% recovery; with sigma = 5 deg you would still nail it to within 2-3%. The law is well-conditioned to fit.

The critical-angle test ran as expected. Sweeping the water-side angle from 0 to 80 degrees, the predicted air-side angle climbs smoothly until it hits 90 degrees at theta_c = 48.75 deg, then becomes complex. No fitting here, just algebra, but it is worth seeing the curve and the boundary on the same axis.

```python
# Fermat: minimize travel time across two media
def travel_time(x):
    return np.hypot(x - xs, ys) + n2 * np.hypot(xd - x, yd)
res = minimize_scalar(travel_time, bounds=(-2, 2), method='bounded',
                      options={'xatol': 1e-9})
x_star = res.x
theta1 = np.arctan2(x_star - xs, ys)
theta2 = np.arctan2(xd - x_star, abs(yd))
print(n1*np.sin(theta1), n2*np.sin(theta2))
```

The Fermat test was the satisfying one. The minimization put the crossing point at x* = 0.2684, with theta1 = 51.75 deg in air and theta2 = 36.19 deg in water. Plugging into the Snell expression:

    n1 sin(theta1) = 1.00 * sin(51.75 deg) = 0.78530
    n2 sin(theta2) = 1.33 * sin(36.19 deg) = 0.78530

The two products agree to 1.2e-8, which is the tolerance of the optimizer. Fermat's principle, applied with no knowledge of Snell's algebra, picks the ray that satisfies Snell's algebra. That convergence is the part that still feels a little uncanny three centuries later: a global minimum of travel time over all possible paths reduces to a local condition on the sine of the angle at the interface.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/55_snell_summary.png)

A small table to compare:

| test | predicted | recovered |
|---|---|---|
| n2 from 20 noisy points (sigma = 1 deg) | 1.330 | 1.333 +/- 0.010 |
| theta_c, water -> air | 48.75 deg | 48.75 deg |
| Fermat optimum vs Snell | identical | diff 1.2e-8 |

## What it may mean

There is no surprise hiding here. Snell's law is one of the few equations in physics that survives essentially intact from its seventeenth-century formulation to current practice. What the exercise may make tangible is how cheaply you can verify a foundational law on a laptop, and how directly two different derivations (Huygens-style wavefronts and Fermat-style action minimization) collapse onto the same number. The Fermat panel in particular is a small instance of a much bigger pattern: most laws of motion and propagation in physics can be written as the stationary point of some integral. Light bending at a water surface, planets tracing geodesics around a star, and electrons spreading through a slit are all members of the same variational family.

The one thing the test does not capture is dispersion. The refractive index depends on wavelength. A real fit on white light through a prism would show separate n values for red and blue (this is what splits sunlight into a rainbow). Snell's law still holds at each wavelength independently; what gets richer is the table.

There is also a quietly nice property of the fit: even when noise pushes individual data points well off the true curve, the least-squares slope through the origin averages those errors down by roughly sqrt(N). With 20 points and 1-degree noise per measurement, the slope error is around 0.7 percent. With Snell's 17th-century protractor and a handful of measurements, the recovered index would have come in noisier (perhaps a few percent), but still well inside the right neighborhood. That is probably part of why the law looked so clean to him: the geometry forgives small angular errors.

## Loose ends

A few honest caveats. I used Gaussian angular noise, which is friendly. Real protractor measurements have heavier tails and a small systematic bias that a calibration step would absorb. Geometric optics breaks down at sub-wavelength scales, where diffraction takes over and the very notion of "angle of refraction" stops being sharp. Polarization changes the amplitudes that reflect versus transmit (Fresnel coefficients), though not the angles themselves. With another afternoon I would refit n versus wavelength on a measured glass-air spectrum and see how well the Cauchy dispersion formula B + C/lambda^2 captures it.
