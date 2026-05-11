# Hooke's 1660 spring law holds to R² = 1.000 in the linear regime and drifts by 28 GPa once the strain crosses 2%.

*Part 56 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1660 Robert Hooke noticed that the more he stretched a wire or a watch spring, the harder it pulled back, and the relationship looked clean. He waited sixteen years before publishing, and then he hid the claim in a Latin anagram: *ceiiinosssttuv*. Two years after that, in 1678, he decoded it: *ut tensio, sic vis*, as the extension, so the force. Three words, one of the oldest quantitative laws in physics, and the seed of everything we now call elasticity theory.

The law in modern dress is F = -k x. Pull a spring by a distance x and it pulls back with a force proportional to x and pointed the other way. The potential energy is ½ k x², a parabola. A mass on the spring oscillates with period 2π √(m/k), which does not depend on how far you pull it. This last property, isochronism, is what makes pendulum clocks tick at a regular rate. Galileo had spotted it in pendulums in 1602; Hooke gave it a force law in 1660; Huygens turned it into a clock in 1673.

The law has held up so well for so long that physicists rarely state what it is not. So I wanted to put numbers on three things at once. How well does a linear fit do in the regime Hooke actually used? How quickly does it fail when you push into anharmonic territory? And does the harmonic-oscillator isochronism still hold when the force law curves?

## What I tried

I synthesized three stress-strain curves on the same strain grid, ε from 0 to 0.20, and fit a linear "modulus" k to each one using only the small-strain window ε ≤ 0.02. The three curves were meant to bracket what Hooke's law can and cannot describe.

The first was an ideal linear spring: σ = E ε with E = 200 GPa, roughly steel. This is the textbook Hooke curve. The fit should be exact.

The second was anharmonic, derived from a Lennard-Jones 6-12 potential around its minimum. LJ is the standard cartoon for an interatomic bond: a sharp repulsive wall on one side, a slow attractive tail on the other, and a minimum somewhere in between. Near the minimum, Taylor-expanding gives F ≈ -k x to leading order with cubic and quartic corrections piling on. Real solids look more like LJ than like an ideal spring; the linear Hooke law is the first term in a series, not the whole story.

The third was bilinear elastic-perfectly-plastic. Linear up to a yield strain ε_y = 0.05, then a flat plateau at constant stress. This is the cartoon of a ductile metal beyond its elastic limit. It is what your paperclip does when you bend it past the spring-back point.

All three curves get the same treatment: fit a single slope and intercept on the small-strain window, then look at the residuals across the whole strain range. The residuals tell you where the linear approximation is good enough and where it is not.

Then I ran a complementary test in the time domain. Pick a mass, give it an initial displacement A, integrate the equation of motion ẍ = F(x)/m, measure the period from the first two upward zero-crossings. Do this for the linear spring and the LJ potential at four amplitudes. If Hooke is exact, the linear period should be identical at every amplitude. The LJ period should not be.

## What happened

The ideal linear fit returned exactly what you would hope. The fitted modulus was 200.000 GPa, the R² on the small-strain window was 1.000000, and the residual across the full strain range maxed out at 7 × 10⁻¹⁵ GPa, which is floating-point dust. There is no signal there; the data is the model.

The anharmonic LJ curve told a different story. The same linear fit on ε ≤ 0.02 returned k = 170 GPa, with R² = 0.997 on the small window. That R² looks reassuring until you ask what happens outside the window. The residual at ε = 0.20 was about 28 GPa, around 14 % of the small-strain stiffness. The residual has a clear shape, growing roughly cubically with strain, exactly what a Taylor series with a nonzero ε³ coefficient produces. The linear approximation is fine for the first few tenths of a percent of strain, decent up to a few percent, and visibly wrong by 10 %.

```python
# fit one linear modulus from the small-strain window
window = eps <= 0.02
k, b = np.polyfit(eps[window], sigma[window], 1)
yhat = k * eps + b
ss_res = np.sum((sigma[window] - yhat[window]) ** 2)
ss_tot = np.sum((sigma[window] - sigma[window].mean()) ** 2)
r2 = 1.0 - ss_res / ss_tot
residual = sigma - yhat   # nonzero residuals outside the window
                          # are the diagnostic that Hooke is breaking down
```

The bilinear curve is the cleanest visual demonstration of where Hooke ends. The linear fit returns k = 200 GPa exactly, R² = 1.000000 on the small window (because the curve is linear there), and then snaps to a 30 GPa residual the moment strain crosses the yield. Below ε_y, Hooke is exact. Above, the residual is the entire plastic flow.

![stress-strain and residuals](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/56_stress_strain.png)

| force law              | fitted k (GPa) | R² (small) | max residual (GPa) |
|------------------------|---------------:|-----------:|-------------------:|
| ideal linear           |         200.00 |   1.000000 |              7e-15 |
| anharmonic (LJ)        |         170.00 |   0.996703 |              27.64 |
| bilinear (yield)       |         200.00 |   1.000000 |              30.00 |

The oscillator test made the same point in time. With m = k = 1, the linear period should be 2π = 6.2832. At four amplitudes spanning two orders of magnitude (A = 0.001, 0.010, 0.050, 0.100), the linear oscillator returned 6.2832, 6.2832, 6.2832, 6.2832. Five-figure isochronism. Hooke and Galileo were not exaggerating.

The LJ oscillator returned 6.137, 6.150, 6.401, 7.011 at the same four amplitudes. At A = 0.001 the period is about 2.3 % short of T₀, because the LJ stiffness near the minimum is not quite the same as the linear fit picked up; at A = 0.10 the period is 11.6 % longer than T₀. This is the standard anharmonic-period correction, and it explains why old pendulum clocks needed the cycloidal cheek or had to be kept on tiny swings. Once amplitude grows, isochronism breaks.

![period vs amplitude](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/56_period_vs_amplitude.png)

## What it may mean

Hooke's law appears to be exactly what a Taylor expansion of any smooth binding potential predicts: a linear restoring force is the first term, accurate near equilibrium, with cubic and higher corrections that bite at large displacement. The numbers above are not new to anyone who has taken a solid mechanics course, but it is satisfying to see the breakdown happen on a single plot rather than as a verbal footnote.

What I had not appreciated before this exercise is how forgiving the small-strain window is. The LJ curve has visible curvature already at ε = 0.005, yet a linear fit there still returns R² = 0.997. If you only ever measure in that regime, you would conclude the material is purely Hookean. This is one reason engineering tables of Young's modulus agree across labs to a few percent: nobody fits modulus on the post-yield regime.

What the oscillator test adds is a dynamical signature that does not depend on careful curve fitting. If you can measure a pendulum or spring's period at two different amplitudes and see no difference at the level your clock can resolve, the restoring force is linear in that range. If you see a 1 % drift between A = 0.01 and A = 0.05, you have detected anharmonicity without ever measuring a force directly. This is, in spirit, what Foucault and his successors did when they used pendulum drift as a probe of Earth physics: time-domain signatures of small force-law corrections.

## Loose ends

The simulation is a cartoon. Real materials have rate dependence, hardening, hysteresis, and temperature-dependent stiffness that none of these toy laws capture. The bilinear yield is perfectly plastic by construction; aluminum hardens, glass shatters, polymers creep. The LJ potential is a fair model for noble-gas crystals and a rough one for almost everything else; for metals you want an embedded-atom potential, for ionic crystals a Born-Mayer form, for polymers something else entirely. With another week I would refit the small-strain window for a real steel tensile-test dataset (NIST has open ones) and see how close the inferred modulus comes to the cataloged 200 GPa, and whether the residual structure looks more cubic or more bilinear. That would tell me whether real metals fail Hooke through smooth anharmonicity or through a sharp yield.
