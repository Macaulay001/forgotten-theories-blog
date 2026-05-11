# Van der Waals' 1873 thesis predicts the CO2 critical point to within a few percent

*Part 71 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Johannes Diderik van der Waals defended his doctoral thesis in Leiden on
14 June 1873. It was called *Over de Continuiteit van den Gas- en
Vloeistoftoestand*, "On the continuity of the gas and liquid states",
and it contained one equation that has outlived almost every other
nineteenth century guess about matter:

  (P + a/v²)(v - b) = RT

Two parameters. One for the attractive pull between molecules, one for
the volume they take up. Plug them in and you get, for free, a critical
point where liquid and vapor become indistinguishable, an S-shaped
isotherm below that point with separate liquid and vapor branches, and
a universal law that should put every fluid onto the same curve when
you rescale by its own critical constants.

Maxwell admired it. Boltzmann thought it was the most important physics
thesis of the decade. Van der Waals got the 1910 Nobel for it. And then
practically every modern textbook moved on to better equations of state.
I wanted to see how well the original held up on a single test case I
could compute in an afternoon.

## What I tried

Carbon dioxide is the canonical example because Andrews had measured
its critical point in 1869 and that work is what van der Waals was
trying to explain. CO₂ has `a = 3.64 L²·atm/mol²` and `b = 0.0427 L/mol`
in the units that make `R = 0.08206 L·atm/(mol·K)` come out clean.

The plan was three steps. First, sweep molar volume `v` across a few
decades and plot the pressure isotherm `P(v) = RT/(v - b) - a/v²` for
five temperatures spanning sub-critical, critical, and super-critical.
Second, locate the critical point analytically (the inflection point
where both `dP/dv` and `d²P/dv²` vanish) and verify it numerically.
Third, overlay isotherms for six gases in reduced coordinates to see
whether the corresponding-states claim actually holds.

The analytic critical point falls out of the algebra in two lines:

  T_c = 8a / (27 R b),    V_c = 3 b,    P_c = a / (27 b²)

Plugging in the CO₂ numbers gives T_c = 307.81 K, V_c = 0.1281 L/mol,
P_c = 73.94 atm. The published experimental values are 304.13 K, 0.0939
L/mol, 72.79 atm. The temperature and pressure are within four percent.
The molar volume is too generous by about 35%.

```python
def vdw_P(v, T, a, b):
    return R*T/(v - b) - a/v**2

Tc = 8*a / (27*R*b)
Vc = 3*b
Pc = a / (27*b**2)
Zc = Pc*Vc / (R*Tc)        # 3/8 exactly, regardless of gas

v = np.linspace(1.05*b, 0.5, 4000)
dP  = np.gradient(vdw_P(v, Tc, a, b), v)
d2P = np.gradient(dP, v)
inflect = v[np.argmin(np.abs(dP) + np.abs(d2P))]
```

That code is the whole experiment. The rest is plotting and bookkeeping.

## What happened

The numerical inflection scan put the critical volume at 0.1282 L/mol
against an analytic 0.1281. They agree to four decimals, which is what
you would hope from a closed-form result, and confirms the calculus is
doing what the algebra promised.

![CO2 vdW isotherms and corresponding-states overlay for six gases](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/71_vdw_isotherms.png)

The left panel shows CO₂ at five temperatures. At 260 K the curve has
the famous S-shape, a local maximum near v = 0.07 L/mol and a local
minimum further out. Between those extrema `dP/dv` is positive, which
is mechanically unstable; in a real experiment the system jumps across
that section at a flat coexistence pressure (Maxwell's construction,
1875). At 280 K the loop has shrunk. At T_c = 307.81 K the inflection
collapses to a point with horizontal tangent. Above T_c the curves are
monotone, which is what "super-critical" means: no phase transition,
just a fluid that gets denser as you compress it.

The CO₂ critical point sits on the figure as a black dot at
(0.128 L/mol, 73.9 atm). Andrews measured the experimental value at
about 304 K and 73 atm. Van der Waals' equation, with parameters
that have no information about the critical point baked in, lands
within roughly four percent on temperature and one percent on pressure.

The compressibility ratio `Z_c = P_c V_c / (R T_c)` is where the model
shows its age. The algebra forces `Z_c = 3/8 = 0.375` for any gas, no
exceptions. Real gases sit between 0.27 and 0.29. Carbon dioxide's
experimental Z_c is 0.274. This is the canonical critique of the vdW
equation, and it is the reason later cubic equations (Redlich-Kwong
in 1949, Peng-Robinson in 1976) replaced it for engineering work.

| quantity     | vdW analytic | CO₂ experimental | error |
|--------------|--------------|-------------------|--------|
| T_c (K)      | 307.81       | 304.13            | +1.2%  |
| P_c (atm)    | 73.94        | 72.79             | +1.6%  |
| V_c (L/mol)  | 0.1281       | 0.0939            | +36%   |
| Z_c          | 0.375        | 0.274             | +37%   |

The right panel tests corresponding states. Take six gases (CO₂, N₂,
O₂, H₂O, Ar, CH₄) and plot each at four reduced temperatures
T_r = T/T_c. If the law of corresponding states holds, all six should
overlay onto one curve at each T_r. They do. The maximum spread in
reduced pressure across the six gases at any reduced volume is about
3 × 10⁻¹⁵, which is floating-point noise. That is not nature being
generous, it is the algebra: once you rewrite the vdW equation in
(P_r, V_r, T_r), the constants `a` and `b` drop out entirely. Every
vdW gas lives on the same dimensionless curve by construction. The
real test would be running this against tabulated experimental data,
which I did not do here, and where the collapse is good but not
perfect.

## What it may mean

A 1873 doctoral thesis predicted the critical temperature and pressure
of CO₂ to within a couple of percent using two empirical constants
fitted from low-density behavior. That is, on this sample of one
substance, a remarkably efficient piece of physics. It also predicted
that all gases should obey one universal equation in reduced
coordinates, and within the model that prediction is exact. Real gases
follow it only approximately, but well enough that engineers still use
corresponding-states correlations to estimate properties of fluids
that have never been measured.

What the vdW equation gets wrong is also instructive. The critical
volume is too large because the simple `v - b` excluded-volume term
overestimates how loosely molecules pack near the critical point. The
compressibility ratio of 3/8 is a hard ceiling that no amount of
parameter tuning can lower. Real molecules attract and repel in ways
that depend on orientation, density, and temperature. Two constants
were never going to capture all of that.

Still, the structure of the answer (critical point, S-shaped loop,
corresponding states) is right. Every cubic equation of state that
followed kept that skeleton and patched the bones. Peng-Robinson, the
workhorse of natural gas processing in 2026, is recognisably a
descendant.

## Loose ends

The Maxwell tie-line construction is the obvious missing piece. With
another afternoon I would compute the equal-area chord on each
sub-critical isotherm and plot the actual coexistence pressure, then
compare to the experimental vapor pressure curve for CO₂ from the
NIST webbook. That would test the equation's most physically
meaningful prediction (the saturated vapor pressure as a function of
temperature) rather than just the critical singularity. The other
loose end is the corresponding-states test on real data: if the
collapse is artificially perfect for the vdW model itself, the
interesting question is how good the collapse is for measured P-V-T
data across many gases. That is a one-week project against the NIST
fluid database.
