# I tested Clapeyron's 1834 vapor-pressure law on water, ethanol, and mercury. One straight line returned each latent heat.

*Part 68 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1834 a French railway engineer named Émile Clapeyron published a
memoir in the *Journal de l'École Polytechnique* trying to make sense
of Sadi Carnot's earlier work on heat engines. Carnot had died of
cholera two years before, and his ideas were drifting toward obscurity
until Clapeyron rewrote them in the language of calculus and gave them
a diagram. Tucked into that paper was a small relation for the slope
of the coexistence curve between a liquid and its vapor. Two decades
later Rudolf Clausius would tidy it into the form modern textbooks use,
combining the ideal-gas law with Clapeyron's identity to get
dP/dT = L·P/(RT²). Integrate that with L treated as constant and you
get something delightful: ln P should be a straight line in 1/T, and
the slope is minus L over R.

What hooked me is that this is a piece of physics from the era before
atoms were taken seriously. Yet it claims you can recover the molar
latent heat of any liquid from nothing but a table of vapor pressures.
No calorimeter. No bomb. Just a ruler on a log plot. I wanted to see
how cleanly that works on real data, and where it should fail.

## What I tried

The plan was small enough to do in an afternoon. Pick three liquids
whose vapor-pressure behavior covers very different chemistry. Water,
the obvious one, with hydrogen bonding and a published latent heat
around 40.7 kJ/mol at the normal boiling point. Ethanol, smaller dipole,
around 38.6 kJ/mol. Mercury, no hydrogen bonding, metallic, around
59 kJ/mol. Hand-encode the vapor pressures from standard tables (IAPWS
steam tables for water, Antoine constants for ethanol, CRC for mercury),
fit ln P against 1/T, and read off L from the slope.

For water I took nineteen points from 0.01 °C up through the critical
point at 374 °C and 22.06 MPa. The pressure ranges from 0.61 kPa near
freezing to 22000 kPa at the critical point, a factor of about 36000.
For ethanol I went from 0 °C to its boiling point at 78.37 °C, where
the pressure hits one atmosphere by construction. For mercury I took
eight points from 20 °C up to its boiling point at 356.73 °C, where
the pressure swings across six orders of magnitude, from 0.17 Pa at
room temperature to 101 kPa at boil.

The water case has a built-in falsification test. Clausius-Clapeyron
assumes there is a latent heat to talk about. Above the critical
temperature the liquid and vapor phases merge into one supercritical
fluid and L collapses toward zero. A linear fit through the sub-critical
data, extrapolated to the critical region, should miss the high-T
points by a visible amount. If the law is real and not just a curve
that bends nicely on log paper, the residuals should diverge in
exactly the right place.

I should be honest about one circularity. The published tables
themselves come from fitted equations of state (IAPWS-IF97 for water,
Antoine for ethanol), so I'm not feeding the analysis truly raw
experimental points. The constants in those equations were originally
calibrated against direct measurements, but a modern table is a
smoothed product. The result is closer to "the textbook smoothing is
consistent with the 1834 law" than "raw thermometer-and-manometer data
confirm the 1834 law".

## What happened

The fit for water (0 to 200 °C) returned a slope of −5083 K. Multiply
by R = 8.314 J/(mol·K) and you get L = 42.27 kJ/mol. R² is 0.99949.
The published L sits between 43.99 kJ/mol at room temperature and
40.65 kJ/mol at the boiling point, so the fitted value is essentially
the average over the range, which is what a constant-L approximation
ought to give.

Here is the core of the regression with the imports stripped out:

```python
T_K = water_T_C + 273.15
x = 1.0 / T_K[T_K <= 473]      # 0-200 C window
y = np.log(water_P_kPa[T_K <= 473])
slope, inter = np.polyfit(x, y, 1)
L_kJ = -slope * 8.314 / 1000.0
# slope = -5083 K, L = 42.27 kJ/mol, R^2 = 0.9995
```

That is the whole analysis. The plot below shows ln P against 1/T for
the three liquids, all three lines remarkably straight across pressure
ranges that span orders of magnitude.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/68_lnP_vs_invT.png)

The numbers for the other two liquids fell into place the same way.

| Liquid | Fit range (°C) | L recovered (kJ/mol) | Published L (kJ/mol) | R² |
|---|---|---|---|---|
| Water | 0–200 | 42.3 | 40.7–44.0 | 0.9995 |
| Ethanol | 0–78.4 | 42.4 | 38.6–42.3 | 0.9999 |
| Mercury | 20–357 | 61.0 | 59.1 | 0.9995 |

Mercury was the one I expected to disappoint. The data span six orders
of magnitude in pressure and a 337 K range in temperature, and mercury's
bonding is nothing like water's. The line stays straight across the
whole table and the recovered L lands within about 3% of the tabulated
59 kJ/mol. That is the part that still feels a bit surprising. A
metal vapor in equilibrium with a liquid metal obeys the same
hydrogen-bond-blind law as water and ethanol because the derivation
never cared about chemistry. It cared only that there is a heat of
transition and an ideal-gas vapor.

For water the supercritical points behaved exactly as predicted. The
sub-critical fit, extrapolated forward, missed the 250, 300, 350 and
374 °C points by progressively larger amounts. By the critical point
the deviation in ln P is about 0.5 nats, a factor of 1.6 in pressure.
The law breaks where it should, near T_c = 647.1 K, in the regime where
the molar volumes of liquid and gas approach each other and the
neglected ΔV liquid term stops being small.

The residuals also show the mild curvature you would expect from
treating L as constant when it isn't quite. For water, L falls from
roughly 45 kJ/mol at 0 °C to 40.7 at 100 °C. That 10% drift over the
fit window translates into a residual pattern with sign changes near
the endpoints. The Watson correlation, an empirical L(T) form from
1943, captures most of that residual; I did not fit it here.

## What it may mean

The thing worth pausing on is how little the equation knows. Clapeyron
wrote it without an atomic theory and without statistical mechanics.
The derivation assumes that the vapor behaves ideally and that the
liquid volume is negligible compared to the gas volume. Both are
approximations. Neither involves any property of the molecules. Yet
the law extracts the correct molar latent heat from vapor-pressure
curves for liquids whose intermolecular forces span hydrogen bonding,
weak dipoles, and metallic cohesion, to within roughly 5% on the data
I looked at.

A useful framing is that Clausius-Clapeyron sits at the boundary
between thermodynamics and kinetics. It tells you the slope of the
phase boundary if you tell it the latent heat, and vice versa. In
practice the same machinery is used today to predict atmospheric
water-vapor content (where the 7% per kelvin scaling of saturation
vapor pressure underlies modern arguments about precipitation intensity
in a warming climate), to design vacuum and cryogenic systems, and to
interpret altitude-dependence of boiling points. The law is not
forgotten in the literal sense. It is in every physical chemistry
textbook. But its 1834 origin and the fact that one straight line
recovers chemistry-blind latent heats is a feature most students meet
once and then stop noticing.

## Loose ends

The published vapor-pressure tables I used are themselves fits, which
weakens the independence claim. A cleaner test would walk back to raw
manometric measurements, ideally from the era when Clapeyron was alive.
I also did not fit a temperature-dependent L; doing so with a Watson
correlation would probably push the water residuals down to numerical
noise. The supercritical breakdown is shown only qualitatively here; a
proper job would model ΔV(T) explicitly and predict the residual
magnitude. Another week and I would also try the law on a polymer
solvent or an ionic liquid to see where its assumptions finally bite.
