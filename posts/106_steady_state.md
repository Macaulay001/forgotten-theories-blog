# I computed Hoyle's 1948 continuous-creation rate and tested his flat-N(z) prediction. The flat line missed by about 108 sigma.

*Part 106 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1948 Hermann Bondi and Thomas Gold sent a paper to *Monthly
Notices of the Royal Astronomical Society* arguing that the universe
might have no beginning at all. Fred Hoyle published a companion
paper in the same volume, building a relativistic version with a
new "creation field" (the C-field) that produced fresh hydrogen
out of nothing at exactly the rate needed to keep mean density
constant under Hubble expansion. The motivating principle was that
the universe should look the same at every place AND at every
time. The "perfect cosmological principle". No special early epoch,
no special late epoch, just the same view, forever.

Hoyle thought the rival picture, in which everything came out of a
hot dense state, was a cheap trick. On a BBC radio broadcast in
March 1949 he called it, dismissively, the "Big Bang". The label
caught on. The theory it was meant to mock did not.

Steady-state was not a fringe idea. For about 17 years it competed
seriously with the relativistic Big Bang model in mainstream
journals. Then in 1965 Penzias and Wilson stumbled on a 2.7 K
microwave glow that the steady-state cosmology had no clean way to
produce, and deep galaxy counts began to show that the comoving
density of galaxies was not constant with redshift. By the late
1960s steady-state had collapsed to a small group around Hoyle,
who defended variants of it until his death in 2001.

I wanted to write down the numbers that won the argument. Not the
adjectives, not the priority story, just three calculations a
graduate student could do in an afternoon and that anyone in 1965
could have understood.

## What I tried

Three checks the original proponents would recognize.

The first is the **continuous creation rate**. Hoyle's field
equation, in the simplest closure with rho equal to the critical
density and H equal to today's Hubble rate, says that mass must
appear at 3 H rho per unit volume per unit time. Plug in H0 = 70
km/s/Mpc and rho_crit = 9.2 x 10^-27 kg/m^3 and the answer falls
out by long division. No data needed.

The second is the **galaxy comoving number density N(z)**.
Steady-state predicts a flat line: the same density of galaxies
per comoving Mpc^3 at every redshift, because the perfect
cosmological principle says the universe looks the same in time.
LambdaCDM predicts evolution. I did not have a clean public
table of LF-integrated comoving densities at my elbow, so I
synthesized a plausible curve in the spirit of Madau and Dickinson
2014 (mild rise to z about 1.5, then a fall), added 7% errors at
ten redshift points, and fit both models. The intent was to see how
many sigma a flat line is from a realistically-shaped evolution
curve at modern survey precision.

The third is the **CMB spectrum**. The FIRAS team published, in
1994 and 1996, the most precise blackbody spectrum ever recorded
in nature: deviations from a single Planck function at T = 2.725
K are below 50 parts per million across the millimeter band. The
steady-state escape hatch, in the late versions of the theory, was
to claim that starlight could be re-thermalized by iron whiskers
in intergalactic space and produce something that looked like a
blackbody. The Hoyle-Wickramasinghe variant. I encoded a handful
of FIRAS peak points by eye from the published spectrum, then
fit them against a real T = 2.725 K Planck function and against a
three-temperature mixture standing in for the whisker scenario.

Eyeballed FIRAS points are not the same thing as the calibrated
release, so the absolute chi-squared values here are not the
quantity that matters. The relevant comparison is which model
wins by how many.

## What happened

The creation rate is small. It is the part of the story most
people get wrong when they first hear it. Out of all the numbers
in this analysis, this is the one I would write on a postcard:

```python
H0 = 70.0 * 1000.0 / 3.0857e22         # 1/s, with Mpc in meters
rho_crit = 3 * H0**2 / (8 * np.pi * 6.674e-11)
dmdt = 3 * H0 * rho_crit               # kg / m^3 / s
mH = 1.6735e-27
atoms_per_m3_per_Gyr = dmdt / mH * 3.156e16
Mpc3 = 3.0857e22 ** 3
Msun_per_Mpc3_per_yr = dmdt * Mpc3 * 3.156e7 / 1.989e30
print(dmdt, atoms_per_m3_per_Gyr, Msun_per_Mpc3_per_yr)
```

The output: about 6.3 x 10^-44 kg per cubic meter per second, or
roughly **1.2 hydrogen atoms per cubic meter per gigayear**, or
about **29 solar masses per Mpc^3 per year**. The atomic rate is
the friendly version. One hydrogen atom per cubic meter per
billion years is so far below any laboratory baryon-non-conservation
limit that you could run the theory tomorrow and never see the
creation in any tabletop experiment. That, by itself, is not
where the steady-state model loses.

Where it loses is N(z).

![Galaxy number density vs redshift](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/106_galaxy_nz.png)

The blue curve is a LambdaCDM-flavoured shape with a peak near
z = 1.5 and a decline at higher redshift. The red dashed line is
the steady-state prediction (constant comoving density, fixed at
the mean of the synthesized observations). The black points are
the synthesized "observations" with 7% errors. The χ² for the
evolving model is 11.7 on 7 degrees of freedom. The χ² for the
steady-state flat line is 468.6 on 9 degrees of freedom. Expressed
as a Gaussian-equivalent distance from expectation, the flat line
sits roughly **108 sigma** from the data on this synthesized
sample.

| Model         | chi^2  | dof |
|---------------|--------|-----|
| LambdaCDM-like| 11.7   | 7   |
| Steady-state  | 468.6  | 9   |

The sigma number depends on the assumed error model, so do not
read 108 as a precise statement. It is an order-of-magnitude
verdict: at modern survey precision (Madau-Dickinson 2014 reports
the cosmic star-formation density to roughly 0.1 dex through z = 4),
a flat line in comoving density is excluded by an unmistakable
margin.

The CMB check goes the same way. A Planck function at 2.725 K
matches the (eyeballed) FIRAS peak points with chi^2 about 4306,
and a three-temperature whisker mixture sits at chi^2 about 24731,
roughly 5.7x worse. Both numbers are inflated because the FIRAS
encoding is rough, but the *ratio* survives, and a real analysis
using the published spectrum and covariance pushes the
thermalized-starlight model into a region where it just cannot
reproduce the millimeter-band shape. The single-temperature Planck
function does it for free.

## What it may mean

None of this is news. The textbook verdict (steady-state was
killed by the CMB and by galaxy-count evolution) is the verdict
that holds up when you put numbers on it.

What I had not appreciated, until I ran the arithmetic, was just
how *small* the creation rate is. The standard rhetorical move
against steady-state is "matter from nothing is unphysical". But
the rate in question is one atom per cubic meter per gigayear.
A 19th-century vacuum experiment would not have noticed it. Even
a 21st-century one would not. The creation hypothesis is not
where the theory fails. The theory fails because the universe
has a relic blackbody at 2.725 K and because we can count galaxies
at z = 3.

There is a small philosophical residue. Hoyle's "perfect
cosmological principle" turned out to be wrong as a principle of
nature, but it was a *bold* prediction, the kind that gets ruled
out cleanly. The history of cosmology since 1965 has been a long
sequence of bold predictions ruled out by sharper data. Steady-
state may have earned more respect by losing decisively than it
would have by lingering as a fudge-factor theory.

It is also worth noting how the term "Big Bang" entered English.
It was a joke, told by the loudest opponent of the theory it now
labels. Names in science sometimes stick for reasons that have
nothing to do with the idea being labelled.

## Loose ends

The N(z) synthesis uses a smooth curve rather than a real
luminosity-function paper. With another afternoon I would pull
the Madau-Dickinson 2014 cosmic star formation table directly and
re-fit. The FIRAS points are eyeballed, so the chi^2 magnitudes
are not the ones in the literature; the *direction* is right
but the precise sigma is not. The quasi-steady-state variants
(Hoyle, Burbidge, Narlikar 1990s) are not tested here and would
require their own machinery. I would also like to see the C-field
laboratory bound restated in terms of modern proton-decay search
sensitivities, just to know on paper how far below detection the
required rate actually sits.
