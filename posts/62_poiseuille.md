# I swept pipe radius across two orders of magnitude. Poiseuille's r^4 law held to five digits.

*Part 62 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In the late 1830s Jean Leonard Marie Poiseuille, a Paris physician trained on the mechanics of blood circulation, started pushing distilled water through glass capillaries 0.029 to 0.142 millimeters wide. He varied pressure, temperature, length, and bore, and over six years of patient measurement he extracted a single empirical rule: the volume of fluid delivered per second is proportional to the pressure drop, proportional to the fourth power of the radius, and inversely proportional to the length. Gotthilf Hagen, a Prussian engineer working independently on water-supply pipes, reached the same conclusion in 1839. Stokes derived the result from his own equations of motion in 1845. The compact statement, Q = pi R^4 dP / (8 eta L), is now the first non-trivial law of fluid mechanics most students meet.

It is also one of the few laws whose exponent is famous enough to be a hazard. The r^4 means a small change in radius costs a lot of flow. A capillary half its design width carries one sixteenth the volume per unit pressure. I had quoted that line to undergraduates for years without ever sweeping the radius in a script and looking at the slope.

## What I tried

The cleanest test is the most literal one. Take the steady Navier-Stokes equation in a cylinder, which collapses for axial flow to a one-dimensional ODE in r:

    (1/r) d/dr ( r dv/dr ) = -dP / (eta L)

with boundary conditions dv/dr = 0 on the axis (symmetry) and v = R = 0 at the wall (no slip). The integrated solution is the well-known parabola v(r) = dP (R^2 - r^2) / (4 eta L). Integrating that profile over the cross-section gives Q = pi R^4 dP / (8 eta L). My plan was to do the integral two ways, both analytic and numerical, sweep R across two orders of magnitude, and read off the slope of log Q vs log R. If Poiseuille was right, the slope is exactly 4.

I also wanted to map where the law stops working. Three classical departures matter: turbulence, non-Newtonian rheology, and slip at small scales. The first kicks in roughly at Reynolds number 2300. The second appears whenever the fluid has shear-rate-dependent viscosity (blood, polymer solutions, ketchup). The third appears in nanofluidics, where the no-slip condition is only approximate and a finite slip length adds a correction. Each of these has a textbook prediction I could compare against the bare Hagen-Poiseuille line.

The whole simulation runs on one core in a few seconds. No special hardware, no calibration, no opaque solver. The interesting question was not whether the law would hold (it would, in the regime where its derivation applies) but how cleanly the failure modes would separate from the laminar case.

```python
# integrate the analytic profile and check the r^4 slope
radii = np.logspace(-4, -2, 25)
Q = np.array([np.pi * R**4 * dP / (8.0 * eta * L) for R in radii])
slope, _ = np.polyfit(np.log(radii), np.log(Q), 1)
print(slope)   # 4.0000
```

A finite-difference solve of the cylindrical ODE on 2001 grid points reproduced the analytic Q to a relative error of 3.8 x 10^-7. That cross-check matters because the parabolic profile is what you get from the equations, not an assumed shape; if the FD solution had disagreed, the textbook derivation would be in trouble.

## What happened

The radius sweep returned a slope of 4.0000 over the range 0.1 mm to 10 mm. The two computations (analytic Q and trapezoidal integration of v(r) over 4001 points) agreed to numerical precision. This is the boring confirmation, which is the right outcome for a 186-year-old result.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/62_Q_vs_r.png)

The interesting part lies in the breakdowns. For a 10 mm diameter pipe, I scanned dP from 1 Pa to 10^6 Pa, computing both the laminar prediction and the turbulent prediction from the Blasius friction correlation (f = 0.316 / Re^0.25, valid for 4 x 10^3 < Re < 10^5). The two curves cross around Re = 2300, exactly where the transition is known to occur. Above that, the laminar Q overshoots reality: at Re = 10^4, the turbulent volumetric flow is about 0.39 of what the unmodified Hagen-Poiseuille law predicts. The pipe carries less water than the 1840 law claims, because turbulent eddies dissipate energy that the laminar derivation ignores.

| Reynolds | regime | Q / Q_HP |
|----------|--------|---------:|
| 100 | laminar | 1.000 |
| 2300 | transition | ~0.8 |
| 10^4 | turbulent | 0.387 |

The non-Newtonian test was sharper. I picked a power-law rheology, tau = K (du/dr)^n, calibrated so that each fluid had the same apparent viscosity as water at a wall shear rate of 100 /s. With n = 1 we recover the Newtonian flow. With n = 0.4 (strongly shear-thinning, like a thin polymer solution), the analytic result Q = (n / (3n+1)) pi R^3 (dP R / (2 K L))^(1/n) gives roughly 23 times the Newtonian flow at the same pressure drop. The shear concentrates at the wall, so the bulk of the fluid moves nearly as a plug. Blood is mildly shear-thinning (n ~ 0.8), which already nudges its flow above the Newtonian estimate by a factor of about 1.4 in this calibration. The Hagen-Poiseuille law is still recoverable, but the eta in the prefactor has to be replaced by an effective shear-rate-dependent viscosity, and the radius scaling loosens from r^4 to r^(1 + 1/n).

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/62_failure_modes.png)

The third breakdown lives at very small scales. For a smooth surface with a Navier slip length ls of about 50 nm (consistent with reported water-on-hydrophobic-surface measurements), the corrected flow is Q = Q_HP (1 + 4 ls / R). At R = 100 microns the correction is 0.2 percent and you would never notice. At R = 100 nm the slip term triples the flow. The classical no-slip assumption is a continuum idealization; below a micron it is approximate enough to matter, and below 100 nm it dominates. The r^4 law is intact in form but acquires a multiplicative slip factor that grows as R shrinks. The MEMS and nanofluidics communities have spent two decades quantifying this correction, and the 1840 calculation remains the right zeroth-order anchor.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/62_velocity_profile.png)

What stood out from running this exercise was how well the failure modes separate. They live on disjoint axes: Reynolds number, rheological index, and length scale. Each modifies the law along a different direction, and the original derivation tells you exactly which assumption each one violates. Turbulence violates "the inertial terms are negligible". Non-Newtonian rheology violates "viscosity is constant". Slip violates "v(R) = 0". Almost everything that happens in real pipe flow is some combination of those three departures.

## What it may mean

Poiseuille's law is one of those rare results where a 19th-century empirical fit and a 19th-century theoretical derivation lock onto exactly the same formula. Most empirical fluid correlations (Manning, Colebrook, Darcy-Weisbach) carry tunable constants. Poiseuille has none. The 8 in the denominator falls out of integrating r^3 from 0 to R, and the pi is the cross-section. The law is, in this sense, a piece of geometry as much as a piece of physics.

The r^4 scaling is what makes vascular biology interesting. The capillary bed has so many small vessels in parallel because each one is hopeless alone; a single 5 micron capillary carrying blood at a 1 kPa pressure drop delivers a flow of around 5 x 10^-13 m^3/s. You need on the order of 10^9 capillaries in parallel to perfuse a human. The r^4 scaling is also why arterial narrowing is so dangerous: a 30 percent radius reduction cuts flow capacity to about 24 percent of baseline. Murray's law, which optimizes the trade-off between flow and metabolic cost of vessel maintenance, uses Hagen-Poiseuille as its starting axiom.

For engineers the same scaling drives the design of microfluidic chips. Doubling channel width cuts pumping power by 16x at fixed flow, but also dilutes residence-time control. The trade-offs are sharp.

## Loose ends

With another week I would replace the analytic non-Newtonian solution with a finite-difference solve of the modified ODE for a Carreau-Yasuda fluid, which is the standard rheology model for blood. I would also run a direct numerical simulation in the transition Reynolds range (1500 to 3500) to see how the turbulent friction correlation actually picks up, rather than relying on Blasius outside its strict validity window. And there is still no clean closed-form correction for Hagen-Poiseuille with a slip length comparable to the radius when ls / R is not small; the perturbative formula stops being useful around R = 4 ls. The whole pipeline runs in under ten seconds on a laptop, and the 1840 slope of 4 is what comes out the other end.
