# Stefan's 1879 T^4 law drops out of one numerical integral, and gives Earth 254.6 K without a single atmosphere line

*Part 57 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Josef Stefan published his radiation law in 1879. It was an empirical
generalization from a handful of cooling experiments (his own, plus
older measurements by Dulong and Petit and by Tyndall): the total
energy a hot body radiates per unit area, per second, grows as the
fourth power of its absolute temperature. Five years later, his
student Ludwig Boltzmann derived the same fourth-power law from
classical thermodynamics combined with Maxwell's radiation pressure.
Write it as

  M(T) = sigma * T^4

with sigma about 5.67 x 10^-8 W per square meter per K^4.

What I find strange about this is the date. 1879. That is 21 years
before Planck's quantum hypothesis. Stefan had no theoretical reason
to expect a fourth power. Boltzmann had a thermodynamic one, but he
also had no spectral law to integrate. They got the right total
without knowing the shape of the spectrum that produces it. Today we
do know the spectrum, Planck's 1900 result, and the question I wanted
to answer was small but irritating: does integrating Planck's law over
wavelength reproduce Stefan's constant exactly? And if so, how close
to zero error can we push with a laptop?

## What I tried

The setup is small. Take Planck's spectral radiance,

  B_lambda(lambda, T) = (2 h c^2 / lambda^5) / (exp(h c / (lambda k T)) - 1)

and integrate it over all wavelengths. Multiply by pi to convert from
radiance per steradian to a hemispherical emissive power. That is
M(T). Stefan's claim is that M(T) is proportional to T^4 with a
universal constant. The proof in textbooks is the substitution
x = h c / (lambda k T), which makes the integrand depend only on x
and pulls a factor of T^4 outside. The remaining definite integral
evaluates to (2 pi^5 / 15) k^4 / (c^2 h^3), and that is sigma.

But that is symbolic. The numerical question is whether quadrature on
a finite grid recovers the same number, and whether it does so across
the very wide range of temperatures where Planck's law is physically
applicable. I picked 50 temperatures from 100 K (a cold planet) to
6000 K (a stellar photosphere), spaced logarithmically. For each T I
integrated B_lambda over a logarithmic wavelength grid spanning three
decades on either side of the Wien peak (lambda_peak = 2.898e-3 / T
meters), with 20000 nodes. Then I fitted log M against log T to
recover both the exponent and the constant.

While I had Planck's law set up I added a second test. Use the Sun's
effective temperature (5772 K) and radius. Compute its total
luminosity by multiplying M(T_sun) by the solar surface area. Spread
that luminosity over a sphere at Earth's orbital distance to get the
solar constant. Then ask the Stefan-Boltzmann law to give back the
temperature at which Earth, treated as a perfect grey sphere with
albedo 0.30, would emit exactly the absorbed sunlight. That is the
classic radiative-equilibrium temperature, and the textbook number is
about 255 K.

## What happened

The core loop is short. The whole computation is a few hundred lines,
but the part that does the actual physics is this:

```python
def planck_lambda(lam, T):
    x = h * c / (lam * kB * T)
    return (2.0 * h * c**2) / lam**5 / np.expm1(x)

def total_emissive_power(T, n=20000):
    lam_peak = 2.898e-3 / T
    lam = np.geomspace(lam_peak / 1000, lam_peak * 1000, n)
    return np.pi * np.trapz(planck_lambda(lam, T), lam)

Ts = np.geomspace(100.0, 6000.0, 50)
Ms = np.array([total_emissive_power(T) for T in Ts])
slope, intercept = np.polyfit(np.log(Ts), np.log(Ms), 1)
```

The fit returned a slope of 4.000000 and an intercept that exponentiates
to sigma = 5.670375 x 10^-8 W/m^2/K^4. The CODATA value is
5.670374 x 10^-8. The relative error is 7 parts in 10^7. On a log-log
plot the 50 points sit on the T^4 line with no visible scatter:

![Stefan-Boltzmann](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/57_stefan_power_vs_T.png)

A few sanity rows from the table:

| T (K) | M(T) numerical (W/m^2) |
|---|---|
| 100 | 5.67 |
| 300 | 459 |
| 1000 | 5.67e4 |
| 3000 | 4.59e6 |
| 5772 | 6.29e7 |

Across nearly six decades in temperature the law holds without
deviation. That is because of the change-of-variable trick. The T
dependence factors out exactly as T^4 before any numerics happen.
Numerical error only enters through the residual definite integral,
which has no T-dependence and is dominated by quadrature accuracy near
the Wien peak. The 20000-node logarithmic grid is overkill; you can
get within 0.1% of sigma with about 200 nodes. Below 100 nodes the
quadrature starts to leak in the long-wavelength tail and sigma drifts
upward by a percent or two.

The Earth calculation came out the same way. M(T_sun) at 5772 K is
6.29 x 10^7 W/m^2. Multiplied by the area of a 6.957 x 10^8 m sphere
that gives a solar luminosity of 3.83 x 10^26 W. Divided by the area
of a 1 AU sphere, the irradiance at Earth's orbit is 1361 W/m^2. With
30% albedo, the equilibrium temperature comes out 254.6 K, against a
textbook airless value of 254.6 K. Same to one decimal.

This is not surprising. The textbook number is computed the same way.
It is reassuring because it means none of the constants slipped: the
solar radius, the AU, sigma, and the albedo all combine through the
balance equation

  pi R_E^2 (1 - A) S = 4 pi R_E^2 sigma T_eq^4

to give T_eq = ((1 - A) S / 4 sigma)^(1/4). The actual Earth surface
sits at about 288 K. The 33 K gap is what the greenhouse effect adds.
That gap is not in this calculation. Nothing here knows what an
atmosphere is.

## What it may mean

The thing worth noticing is how much physics Boltzmann got right
without a spectral law. He had thermodynamics, Maxwell's stress
tensor for the radiation field, and the Carnot-style argument that a
cavity in equilibrium with itself can be reversibly expanded. From
that he proved the fourth power and even fixed sigma up to an
undetermined integration constant. Planck's later work supplied that
constant and revealed the spectrum, but the integrated law was
already settled. In the data here, the T^4 scaling is perfect because
Planck's law is built from a Bose-Einstein-like denominator whose only
T dependence enters through the dimensionless combination
hc / (lambda kB T). Any spectral distribution with that property gives
a T^4 integral. Stefan-Boltzmann is therefore a much weaker statement
than Planck's law: it constrains a single moment of the spectrum,
nothing else.

The 254.6 K answer for Earth is on the same footing. It assumes the
planet is an isothermal black body in radiative equilibrium with the
Sun. The number is insensitive to small changes in albedo (a 0.05
change in A shifts T_eq by about 5 K) and almost insensitive to
anything else. What it does not include is the atmosphere, the
rotation, the oceans, or the seasons. Those raise the actual mean
surface temperature by about 33 K. Whether 33 K is stable or fragile
depends on the optical depth of every absorbing band in the atmosphere,
and that is the part the simple law cannot address.

## Loose ends

The exponent recovery is so clean because Planck's law contains the
T^4 scaling analytically. A more interesting test would be to fit a
non-Planckian emitter, for instance a real star with spectral lines
and limb darkening, and see how much the recovered "sigma" drifts
when you treat it as a black body. With another week I would pull a
calibrated solar spectrum from SORCE or TSIS-1 and ask whether the
integrated solar irradiance matches sigma * T_eff^4 to better than 1%.
My guess is yes, within the published 0.05% uncertainty on TSI, but I
have not done the integral.
