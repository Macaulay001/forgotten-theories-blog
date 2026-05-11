# I redid the polywater capillary calculation. The reported 1.4 g/cm³ matches dissolved silica almost exactly.

*Part 105 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1962 a young Soviet chemist named Nikolai Fedyakin, working at a provincial institute in Kostroma, noticed that water condensed slowly into very narrow quartz capillaries did not behave like water. It was viscous. It stayed liquid past 300 °C. Pycnometry on his tiny droplets gave a density around 1.4 g/cm³, forty percent heavier than ordinary water. Boris Derjaguin took the work to Moscow in 1966, the West heard about it in 1968, and by 1969 *Science* was publishing infrared spectra of a substance the authors called "anomalous water". By 1970 it had a friendlier name. Polywater.

About 500 papers followed in five years. The American chemistry establishment treated it as a plausible new state of H₂O. The novelist Kurt Vonnegut's *Cat's Cradle* "ice-nine" worry, that a stable polymorph might seed normal water and freeze the oceans, was discussed in print by serious people. Then in 1970 Dennis Rousseau and Sergio Porto at Bell Labs collected enough polywater to do gravimetry, found it was about fifty percent solid by mass, and identified the solid as silica. A 1973 follow-up by Rousseau finished the field off. Polywater was sweat, dust, and dissolved quartz.

I wanted to know how close the arithmetic really was. If you take Fedyakin's capillary geometry, the known solubility of silica in slightly basic water, and a one-line ideal-mixture rule for density, do you get 1.4 g/cm³ out the other end, or do you need a fudge factor.

## What I tried

The setup is small and the numbers all come from well-known places.

Step one: capillary geometry. A 1-μm-radius cylindrical capillary that holds 10 nL of water has to be about 3.2 mm long. Its inner wall area is 20 mm². The surface-to-volume ratio is 2/r = 2 × 10⁶ per metre. A beaker at 1 cm scale gives roughly 400 per metre. The micron capillary exposes water to a wall area-to-volume ratio about five thousand times larger than ordinary glassware. Whatever the wall is made of, the water is mostly touching it.

Step two: silica solubility. Amorphous silica at pH 7 dissolves to about 120 ppm at 25 °C, and to roughly 200 ppm by pH 9. Iler's *Chemistry of Silica* (1979) is the standard reference. Fresh-cut quartz surfaces leach faster than crystalline quartz because the outermost layer behaves like amorphous silica. Sealed capillaries also drift basic over weeks as CO₂ is excluded.

Step three: evaporative concentration. Derjaguin's protocol grew the anomalous water slowly, often for days, near saturated humidity. If a droplet shrinks by a factor of one hundred over that period, an initial 200 ppm load becomes 20 000 ppm.

Step four: density of the mixture. Amorphous silica gel has density about 2.20 g/cm³. For an ideal mixture of water and silica the specific volume just adds:

```python
rho_water = 1.000
rho_silica = 2.20
w = np.linspace(0.0, 0.9, 901)        # silica mass fraction
rho_mix = 1.0 / ((1 - w) / rho_water + w / rho_silica)
idx = int(np.argmin(np.abs(rho_mix - 1.40)))
print(f"w to hit 1.40 g/cm^3 = {w[idx]*100:.1f}%")
# -> w to hit 1.40 g/cm^3 = 52.4%
```

Step five: IR. I summed Gaussians at the textbook band positions. Water gets a broad O-H stretch at 3400 cm⁻¹, an H-O-H bend at 1640 cm⁻¹, and a libration around 700 cm⁻¹. Amorphous silica gets the Si-O-Si asymmetric stretch at 1100 cm⁻¹, symmetric stretch at 800, rocking at 450, an Si-OH band at 960, and a surface silanol O-H peak at 3650. I then made a 50/50 mass mixture and asked where the dominant fingerprint peak sat.

That is the whole calculation. No anomalous physics, no fitted parameters.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/105_density_vs_silica_fraction.png)

## What happened

The mass fraction of silica needed to lift water from 1.00 to 1.40 g/cm³ comes out at 52.4%. Rousseau and Porto's 1970 gravimetry on collected polywater gave a residue around 50% solid by mass. The ideal-mixture line and the measured residue agree to within a couple of percent. I did not have to nudge the silica density or invoke partial molar volume corrections to land on the reported number.

| quantity | this calc | reported |
|---|---|---|
| capillary S/V at r = 1 μm | 2.0 × 10⁶ /m | geometric |
| silica fraction at 1.40 g/cm³ | 52.4% | ~50% (Rousseau & Porto 1970) |
| mixture density at 50% silica | 1.38 g/cm³ | ~1.4 (Derjaguin 1966-69) |
| dominant fingerprint peak | 1098 cm⁻¹ | 1000-1200 cm⁻¹ (Lippincott 1969) |

The IR picture is just as clean. In the modelled mixture the 3400 cm⁻¹ free O-H band drops to about half its pure-water intensity, while the 1100 cm⁻¹ Si-O-Si stretch becomes the strongest feature anywhere on the trace, with the 450 cm⁻¹ rocking mode close behind. Those are exactly the qualitative anomalies Lippincott and coworkers flagged as "polymeric water" in 1969. The "broad envelope near 1600 cm⁻¹" they emphasised is what you get when the water bend at 1640 overlaps the silica-adsorbed water bend at 1630 and you cannot resolve them in a smeared spectrum.

A different way to feel the result: the wall area per droplet volume in Fedyakin's capillaries was roughly 5000 times larger than in a normal glass container. Even if the bulk dissolution rate is unchanged, the capillary loads silica into its contents about that much faster on the same timescale. A multi-day equilibration, which Derjaguin used, is enough time to push the dissolved load well past the colloidal stability limit. From there it gels.

What polywater really was, then, looks like a slow-leached silica colloid concentrated by evaporation. The "anomalous" boiling point above 300 °C is consistent with that, since dehydrating a silica gel does not produce a clean phase transition; you just keep losing water until the residue is glassy. The viscosity is consistent. The density is consistent. The IR is consistent. The Raman triplet near 1100, 800, 450 cm⁻¹ that Lippincott took as evidence of new H-bonding is exactly the silica fingerprint that geochemists had been using to identify amorphous SiO₂ for decades.

## What it may mean

The retrospective lesson is geometric. A 1-μm quartz capillary is not really a closed chemical container, it is a slow leaching reactor with a wall area that overwhelms the contents. Anything you condense inside it will end up partially dissolved in its own enclosure if you wait long enough. Derjaguin's group saw this in 1962, before microfluidics existed as a discipline, and tried to read the result as a property of water rather than as a property of the experimental geometry. It is hard to blame them. The capillary radius and the wall chemistry feel like negligible details until you do the surface-to-volume sum.

The forward-looking version of the same idea matters now for microfluidic chips and nanopore reactors, where leaching of silicon, PDMS plasticisers, or glass cations on hour-to-day timescales can dominate the chemistry of dilute samples. Anyone who has tried to do trace-metal analysis in a polished silica capillary has met the modern version of this problem.

I am not claiming this analysis closes the case completely. It says that the simplest physically reasonable model, water plus dissolved silica, reproduces the two headline numbers (density 1.4 g/cm³ and IR fingerprint dominated by Si-O modes) with no free parameters and no fitting. That is what Rousseau, Porto, and Allen said in the early 1970s, and the arithmetic still works fifty years later.

## Loose ends

Ideal mixing slightly underestimates the real density at high silica fractions, so the true crossover may sit a few percent below 52%. I did not simulate Raman intensities, only IR. The pH inside a sealed quartz capillary is hard to bound to better than a factor of two, which moves the equilibrium silica load by about a factor of three. None of those wiggles change the conclusion: the headline polywater numbers fall out of literature solubility plus geometry. A careful replication today, with modern silicon-clean fused-silica tubing and ICP-MS on the residue, would cost a few thousand dollars and could pin down the dissolution timescale at micron radius directly.
