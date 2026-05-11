# I fit V = I R to four devices on the same voltage sweep. Only one of them stayed linear.

*Part 65 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Georg Ohm published *Die galvanische Kette, mathematisch bearbeitet* in
Berlin in 1827. The book argued that current through a metallic
conductor is proportional to the voltage across it, with a material
constant called resistance. The result was simple enough to write on a
napkin (V = I R) and unexpected enough that the Prussian minister of
education called it "a web of naked fancies" and Ohm lost his teaching
post. By the 1840s Kirchhoff and Wheatstone had built the early circuit
theory on top of it, and the napkin equation became the first thing a
physics student learns in week one.

It has always been a material-specific statement. A wire of copper
obeys it across many decades of current. A silicon diode does not. A
hot tungsten filament does, but only when it is cold. I wanted to see
the gap between "Ohm's law works" and "Ohm's law fails" as numbers, on
a single voltage sweep, with the same fit applied to each device.

## What I tried

The plan: synthesize four V-I curves on shared voltage grids,
representing the four classic device families that show up in any
undergraduate electronics text.

The ohmic resistor was the control. Current I = V / R with R = 100 Ω,
plus Gaussian measurement noise of σ = 10⁻⁴ A. If the linear fit
recovers R_true and produces white residuals, the test framework works.

The silicon diode used the Shockley equation, I = I_s (exp(V / (n V_T))
− 1), with I_s = 10⁻¹² A and V_T ≈ 25.85 mV at 300 K. This is the
exact replacement equation Shockley wrote in 1949 to describe p-n
junctions. The current rises exponentially past roughly 0.6 V and a
linear fit has no real meaning anywhere on the curve.

The tungsten filament needed a self-heating model. Tungsten's
resistivity rises with temperature, R(T) = R₀ (1 + α (T − T₀)) with
α = 4.5 × 10⁻³ /K. At steady state the electrical power V²/R(T)
equals the radiative loss ε σ A (T⁴ − T₀⁴). I solved that balance
numerically for each applied voltage. At low V the filament stays
near room temperature and looks ohmic; as V grows the operating
temperature climbs toward 2500 K and R triples or quadruples.

The NTC thermistor used the opposite physics. Its resistance falls
with temperature as R(T) = R_ref exp(β (1/T − 1/T_ref)) with β =
3500 K. Self-heating against Newtonian cooling k (T − T_amb) gives a
steady-state temperature for each V, and the curve bends in the
opposite sense from the filament: more voltage means lower R, means
more current per added volt.

For each device I fit V = I R on a small low-voltage window, then
checked the residual across the entire sweep. The break voltage
v_break is the first V past the fit window where the residual
exceeds 5 percent of the local current. This is a rough yardstick,
not a textbook criterion. The point was to pin a single number on
where the linear approximation gives up.

## What happened

The ohmic resistor returned R_fit = 99.9 Ω against R_true = 100 Ω,
with R² = 1.0000 over the entire 0 to 5 V sweep. The residuals were
white Gaussian, as expected. No v_break: the linear approximation
holds throughout.

The Shockley diode broke immediately. Below 0.3 V the diode current
is microscopic (~10⁻⁹ A at 0.3 V, ~10⁻⁵ A at 0.5 V, ~10⁻¹ A at
0.7 V). Fitting V = I R on the 0 to 0.3 V window gave a nominal
R_fit ≈ 6.4 MΩ with R² = 0.43, mostly meaningless because the
current is dominated by exponential growth that the fit cannot
capture. By 0.7 V the actual device current is 7 orders of
magnitude above the linear extrapolation.

The tungsten filament was the interesting case. The fit on the 0 to
1 V window gave R_fit = 12.6 Ω, close to the cold resistance R₀ =
10 Ω. R² = 0.9939 over the low window. Past 1 V the curve bends
visibly: at 12 V the model temperature is around 2500 K and the
filament's hot resistance is roughly 3x its cold value. The break
voltage came in at 1.03 V, right at the edge of the fit window.
Above that the linear extrapolation overshoots the measured current
by tens of percent.

The NTC thermistor was the cleanest illustration of "ohmic in the
small-signal limit, non-ohmic when self-heating matters". Below 0.3 V
the thermistor sits near ambient temperature and the fit recovered
R_fit = 994 Ω against R_ref = 1000 Ω at R² = 1.0000. The break came
at v_break = 0.653 V. Past that, the operating temperature climbs and
R drops, so the current bends upward away from the linear
extrapolation.

| device              | fit window (V) | R_fit (Ω) | R²      | v_break (V) |
|---------------------|---------------:|----------:|--------:|------------:|
| ohmic resistor      | 0 to 5.0       | 99.9      | 1.0000  | none        |
| Si diode (Shockley) | 0 to 0.3       | 6.43e6    | 0.4315  | 0.302       |
| tungsten filament   | 0 to 1.0       | 12.6      | 0.9939  | 1.03        |
| NTC thermistor      | 0 to 0.3       | 994       | 1.0000  | 0.653       |

![V-I curves and residuals from a linear fit](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/65_vi_panel.png)

The figure shows the four curves on the top row with their linear fits
as dashed lines and the fit windows shaded. The bottom row is the
residual I_measured − I_fit on the same voltage axis. The ohmic
residual is featureless noise. The diode residual is dominated by the
exponential climb past the fit window and dwarfs the plotted current.
The filament residual bows downward (current grows slower than linear
because resistance grew with temperature). The thermistor residual
bows upward (current grows faster than linear because resistance fell).

The core of the simulation was a small numerical balance for each
self-heating device. For the filament:

```python
# steady-state temperature: electrical input = radiative loss
def filament_current(V):
    def balance(T):
        R_T = R0 * (1.0 + alpha * (T - T0))
        P_elec = V * V / R_T
        P_rad  = eps_sigma_A * (T**4 - T0**4)
        return P_elec - P_rad
    T_eq = brentq(balance, T0 + 0.01, 3500.0)
    R_T = R0 * (1.0 + alpha * (T_eq - T0))
    return V / R_T
```

Brent's method solves the energy balance in a few iterations per
voltage step. The same template, with a different R(T) law and
cooling term, gives the thermistor. The diode and ohmic resistor are
closed-form.

## What it may mean

The standard reading is that Ohm's law is a constitutive relation,
true for some materials and false for others. The simulation gives a
sharper version of that statement. Every device in this set is
roughly ohmic somewhere. The interesting parameter is the voltage
window over which V = I R holds to within a few percent, and that
window is set by the underlying device physics: thermal voltage for
the diode, thermal time-constant times radiative loss for the
filament, the β coefficient for the thermistor. Once you know v_break,
the linear law is a useful first approximation; once you cross it,
the device equation takes over.

This may also explain why Ohm's contemporaries pushed back. In 1827
the cleanest available conductors were not always clean. Wires
self-heated, electrolytes polarized, contacts oxidized. The
non-ohmic catalogue we now teach as exceptions was, for the early
electricians, the typical case. Ohm's contribution was isolating the
regime where the linear law held, then trusting it.

What we can't rule out from a synthesized study is how well the
specific device models match real measurements. The Shockley equation
omits series resistance and recombination current. The filament model
omits lead conduction and emissivity variation with T. The thermistor
model treats k as constant. The qualitative break is correct; the
exact v_break numbers would shift on real hardware.

## Loose ends

With another week and a bench, I would replicate the four curves on
a programmable source-meter, a 100 Ω metal-film resistor, a 1N4148
diode, an automotive 12 V incandescent bulb, and a 10 kΩ NTC bead.
Sweeping at a few mA per second and logging V and I would give real
v_break values to compare against the simulated ones. The full
synthesis runs in under a second on a laptop, and the conclusion of
the small case has been stable for 199 years.
