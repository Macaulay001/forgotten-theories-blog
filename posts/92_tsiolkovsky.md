# Tsiolkovsky's 1903 logarithm still draws the wall for chemical rockets

*Part 92 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Konstantin Tsiolkovsky was a deaf schoolteacher in Kaluga who had
been writing about space travel since the 1880s. In 1903, in a small
Russian journal called *Nauchnoe Obozrenie*, he published the
equation that every aerospace student now recites in their sleep:

    Δv = v_e · ln(m_0 / m_f)

The velocity a rocket can gain equals its exhaust velocity times the
natural log of the wet-to-dry mass ratio. The logarithm is the
killer. It says you cannot brute-force your way to higher Δv by
adding propellant linearly; the propellant has to lift its own
propellant, and the cost compounds exponentially in mass.

The equation sat almost unnoticed for two decades. Robert Goddard
rediscovered the same logic independently in 1912. Hermann Oberth
worked it out again in Germany in the 1920s. By the time von Braun's
team was sizing the Saturn V, the equation was lore. It is also one
of the few results in physics where the original paper, the textbook
version and a Python one-liner all agree to five decimal places.

I wanted to push the equation against the missions people actually
care about, and see how much room chemistry leaves us.

## What I tried

The plan is small. Pick a handful of standard mission Δv targets:
9400 m/s to climb out of Earth's well into low orbit, 3900 m/s to
push from LEO to geostationary, 3200 m/s for a translunar injection,
3600 m/s for a Mars transfer, and 13600 m/s for the cumulative
surface-to-Mars-landing budget. These come from the canonical
astrodynamics tables and are within a few hundred m/s of what
anyone's mission planner would quote.

For each Δv, compute the required mass ratio m_0/m_f for three
propulsion classes:

- chemical, Isp ≈ 450 s, which is roughly LH2/LOX upper stages
- nuclear thermal, Isp ≈ 900 s, the NERVA-class regime
- ion or Hall, Isp ≈ 3000 s, deep-space electric

The mass ratio R = exp(Δv / (g0·Isp)). That is the whole equation.
The interesting question is what R does to the payload fraction once
you nail down a structural fraction ε, the ratio of dry-stage mass
to wet-stage mass. With payload p on top, the wet mass m_0 satisfies
m_0 = R · (ε · m_0 + p), so p/m_0 = 1/R − ε.

That subtraction is where staging earns its keep. If R · ε ≥ 1 the
single-stage solution diverges: no amount of propellant gets you
there, because the propellant tank is heavier than what you can lift
with the rest of the propellant. Splitting Δv across two stages cuts
each stage's R, and the diverging combination of R and ε comes back
under control.

I also wanted to be honest about where the equation lies to us. Ion
thrusters can technically supply 3000 s Isp, but their thrust is
millinewtons per kilowatt. Plugging Isp = 3000 s into a
surface-to-orbit Δv budget gives a beautifully small mass ratio of
1.38, which is mathematically correct and operationally meaningless.
The rocket would not lift off. The Tsiolkovsky equation is a
necessary condition, never a sufficient one.

## What happened

The numerics reproduce the textbook numbers to the second decimal.

Surface to LEO with chemical Isp: m_0/m_f = exp(9400 / (9.80665 · 450))
= 8.42. That is essentially the Saturn V first-stage ratio, the
SpaceX Falcon 9 ratio, the Soyuz ratio. Every operational launcher
sits within roughly 10% of this number because the equation will not
let them sit elsewhere.

Switch to nuclear thermal at Isp = 900 s and the same 9400 m/s costs
a mass ratio of 2.90. The Mars-landing cumulative Δv of 13.6 km/s
falls from 21.8 (chemical, basically impossible single-stage) to
4.67 (nuclear thermal, comfortable) to 1.59 (ion, trivial on paper).

The staging test was the cleanest finding. Hold ε = 0.10 (a generous
upper-stage structural fraction) and ask for 9400 m/s with Isp = 450
s. The single-stage initial-to-payload ratio comes out at 53.1. Split
the Δv evenly across two stages and the same ratio drops to 16.7.
The factor is 3.18x: same propellant, same engines, same destination,
three times less wet mass at the pad. The Tsiolkovsky equation is
multiplicative across stages but additive in Δv, so two ln(R_1 + R_2)
stages always beat one ln(R_1 · R_2) stage as long as the structural
penalty for adding a stage is not absurd.

Here is the core, stripped of bookkeeping:

```python
G0 = 9.80665

def mass_ratio(dv, isp):
    return np.exp(dv / (G0 * isp))

def single_stage_initial(dv, isp, payload=1.0, eps=0.10):
    R = mass_ratio(dv, isp)
    return R * payload / (1.0 - R * eps)  # diverges when R*eps >= 1

def two_stage_initial(dv, isp, payload=1.0, eps=0.10):
    dv_each = dv / 2
    m2 = single_stage_initial(dv_each, isp, payload, eps)
    return single_stage_initial(dv_each, isp, m2, eps)
```

A small results table:

| Mission                       | Chem (450 s) | NTR (900 s) | Ion (3000 s) |
|-------------------------------|-------------:|------------:|-------------:|
| LEO from surface (9.4 km/s)   |         8.42 |        2.90 |         1.38 |
| LEO → GEO (3.9 km/s)          |         2.42 |        1.56 |         1.14 |
| LEO → Mars transfer (3.6 km/s)|         2.26 |        1.50 |         1.13 |
| Surface → Mars landing (13.6) |        21.80 |        4.67 |         1.59 |

The sensitivity to structural fraction is sharper than I expected.
At ε = 0.08 the single-stage Δv = 9400 m/s vehicle has wet-to-payload
ratio 25.8 and the two-stage has 14.3, a 1.8x gap. At ε = 0.10 the
gap is 3.18x. At ε = 0.12 the single-stage version mathematically
diverges (R · ε > 1) while the two-stage still sits at ~20. The
benefit of staging is small for light tanks and apocalyptic for
heavy ones. That is the entire history of upper-stage engineering in
one sentence.

![mass_ratio_vs_dv](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/92_mass_ratio_vs_dv.png)

On the log plot the three Isp curves are straight lines with very
different slopes. Chemical climbs through ratio 20 at about 13.5
km/s, which is why surface-to-Mars in one chemical stage is a
non-starter. Nuclear thermal stays under 5 across every realistic
mission. Ion barely leaves 1, but its time cost (months of thrust)
keeps it pinned to robotic deep-space work.

## What it may mean

Tsiolkovsky's logarithm appears to behave less like a guideline and
more like a hard ledger. The reason in-orbit refueling, aerobraking,
and gravity assists exist as a research program is that they let
mission designers shave 1-3 km/s off a Δv budget, which inside an
exponential is the difference between a launchable vehicle and a
spreadsheet entry. Falcon 9 sea-level Isp is around 282 s; plug that
in for the first stage and the LEO mass ratio rises from 8.4 toward
30, which is why the booster's structural fraction has to be brutal
(near 0.04) for the design to close. Tsiolkovsky pays no attention
to whether the engine is reusable, autogenously pressurized or
3D-printed. It just asks for the mass ratio and waits.

The other thing the test makes vivid is that the equation rewards Isp
asymmetrically. Doubling chemical Isp (which is essentially impossible
with chemistry; LH2/LOX is already near the theoretical ceiling) would
turn the Mars-landing ratio from 21.8 into 4.67. That is the prize on
the other side of nuclear thermal, and it is why NASA keeps revisiting
NTR every few decades. The physics of getting that factor of 2 in
Isp is not budget-bound; it is institutional.

## Loose ends

I used a single structural fraction across stages and an even Δv
split. Real designs solve a small optimization with stage-dependent
ε and Isp, sometimes splitting 60/40. Δv budgets here exclude
gravity losses (typically 1-1.5 km/s) and steering losses (a few
hundred m/s). The "ion to LEO" entry is mathematical only; electric
thrusters cannot lift their own weight. With another afternoon I
would refit the staging optimum across asymmetric splits and check
how much the 3.18x saving moves; I would guess up to 3.5x with
optimal allocation, but the qualitative wall stays where it is.
