# I fitted Beer's 1852 absorption law to a simulated methylene-blue spectrometer. The slope came back at 85,102 against an 85,000 target.

*Part 60 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1729 the French optical engineer Pierre Bouguer cut equal slabs of glass and noticed that each one absorbed the same fraction of a candle's light. Three slabs reduced the brightness by the cube of the single-slab fraction. Lambert, a generation later, wrote down the exponential decay this implied. Then in 1852 August Beer, working in Bonn on colored solutions, extended the same exponential to concentration: the more dye you add to the cuvette, the more light gets eaten, and the eating scales the same way as path length. The combined law, written in modern form, is A = epsilon c L, where A is the base-10 absorbance log10(I0/I), epsilon is the molar absorptivity in L mol^-1 cm^-1, c is concentration, and L is path length.

This is the equation in every analytical chemistry textbook. It runs spectrophotometers, pulse oximeters, and most colorimetric assays. It is also, like every clean physics law, an approximation. Real dyes aggregate at high concentration. Turbid samples scatter. Detectors saturate. I wanted to map exactly where the approximation holds and where it slips, on a known system, with synthetic data so the ground truth was never in doubt.

## What I tried

The plan was to simulate a UV-visible spectrometer measuring methylene blue at 664 nm, the wavelength of its main visible absorption peak. Methylene blue has a literature molar absorptivity around 85,000 L mol^-1 cm^-1 in dilute aqueous solution, which is high enough that a 1 cm cuvette absorbs noticeably at micromolar concentration. I set that as the ground truth and asked three questions in sequence.

First, can a noisy fit recover epsilon in the linear regime? I generated 25 transmitted-intensity values for concentrations evenly spaced from 0 to 15 uM, applied Gaussian noise at 0.5% of the signal, computed A = log10(I0/I), and fitted a line.

Second, what happens when the dye starts forming dimers? Methylene blue is famously prone to self-association. Two monomers (M) collapse into a dimer (D) with an equilibrium constant K = [D]/[M]^2 around 3.8 x 10^3 M^-1 in water at room temperature. The dimer has a different absorptivity at 664 nm, lower per monomer unit by roughly 30%, because some of the oscillator strength shifts to a satellite peak near 605 nm. I solved the quadratic 2K[M]^2 + [M] - c_total = 0 for the monomer fraction at each total concentration, computed an apparent absorbance from the weighted sum of monomer and dimer contributions, and added a small turbidity-like offset to mimic light scattering at the highest concentrations.

Third, does the law stay additive for a two-component mixture? I built a synthetic dye B with a Gaussian absorption band centered at 540 nm and a peak epsilon of 90,000, and asked whether the absorbance of a mixture equals the sum of the two pure-component absorbances at every wavelength, the way Beer-Lambert says it should.

The whole pipeline is one Python file. The cuvette is a single line of arithmetic.

## What happened

Phase 1 was essentially a sanity check, and Beer's law passed it cleanly. The 25-point linear fit returned epsilon = 85,102 L mol^-1 cm^-1 against the 85,000 ground truth, an error of 0.12%. The coefficient of determination was 0.99998. Residuals stayed within plus-or-minus 0.01 absorbance units across the full range, with no visible curvature. At 0.5% detector noise, a couple of dozen carefully spaced points is enough to nail epsilon to roughly four significant figures.

![Linear-regime fit and residuals](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/60_linear_fit.png)

Phase 2 is where the picture gets interesting. The ideal Beer-Lambert prediction climbs linearly through six decades of concentration. The apparent absorbance follows it for the first four, then bends downward. By 10^-4 M the apparent epsilon (the ratio A/(cL) that a careless analyst would extract) has dropped from 85,000 to about 76,000. By 5 x 10^-4 M it has fallen to roughly 70,000, an 18% departure from the dilute-limit value. The dimer fraction at 5 x 10^-4 M total dye is around 40%, and because the dimer has a lower per-monomer absorptivity at 664 nm, the apparent extinction sags.

```python
# methylene blue monomer-dimer equilibrium
def monomer_conc(c_total, K):
    a, b, c = 2*K, 1.0, -c_total
    return (-b + np.sqrt(b*b - 4*a*c)) / (2*a)

M = monomer_conc(c_sat, K_DIM)
D = (c_sat - M) / 2.0
A_apparent = (EPS_M * M + EPS_DIMER_PER_MONOMER * D) * L
eps_apparent = A_apparent / (c_sat * L)
```

The same effect is responsible for one of the standard headaches in spectrophotometric assays. If you calibrate at sub-micromolar concentrations and try to use the calibration at hundred-micromolar, your readings come back biased low and you under-report the analyte. Most operating manuals tell you to dilute aggressively; the dilution rule is exactly the warning that you should sit deep inside the linear regime, far below the aggregation threshold.

![Saturation due to monomer-dimer equilibrium](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/60_saturation.png)

Phase 3 confirmed additivity with room to spare. The simulated mixture spectrum, at 10 uM dye A plus 15 uM dye B, matched the algebraic sum of the two single-component spectra to within 0.0071 absorbance units at the noisiest wavelength. That worst-case error is mostly detector noise, not a structural breakdown. As long as the two dyes do not chemically interact, Beer-Lambert's prediction that absorbances add holds across the visible band.

| regime | A_ideal | A_apparent | apparent eps (L/mol/cm) |
|--------|--------:|-----------:|------------------------:|
| 1 uM | 0.085 | 0.085 | 85,001 |
| 100 uM | 8.50 | 7.79 | 77,900 |
| 500 uM | 42.6 | 34.9 | 69,800 |

The third row's absorbances are physically unmeasurable on most instruments (a real detector would saturate above A ~ 3) but the algebraic comparison still tells you exactly where the linear extrapolation would have led you astray, and by how much.

The detection-limit side calculation came out at roughly 77 nM. At 0.5% relative noise on the transmitted intensity, the noise on absorbance is sigma_A = 0.005 / ln(10) ~ 0.00217. A 3-sigma minimum signal corresponds to A_min ~ 0.0065, and with epsilon L = 85,000, this gives c_min ~ 77 nM. Real visible spectrophotometers do roughly this well on a strongly-absorbing dye in a clean buffer.

## What it may mean

What strikes me, after running this, is how generous the linear law is. Bouguer chose his glass slabs more than 250 years before anyone could measure transmittance to 0.1%, and his linear-extinction picture survives translation into a precise modern fit with a sub-percent residual. The law works because, at low optical density, every photon's chance of being absorbed is independent of every other photon's, and the absorbers are too dilute to see each other. Those two assumptions are tight enough that the linear regime sits comfortably across roughly three orders of magnitude in concentration for a typical visible dye.

The departures, on this sample, look exactly as the chemistry literature has been describing them since the 1950s. Dimerization bends the absorbance curve sub-linearly, and the bend grows monotonically with concentration. A simple two-state equilibrium with one rate constant and one altered absorptivity captures most of the shape. More elaborate aggregation (trimers, micelles, hydrogen-bonded clusters) would add curvature on top, and ionic strength, temperature, and pH all shift K_dim. None of this overturns Beer-Lambert; it relocates the boundary of the linear envelope.

The additivity result might be the most useful operational point. Pulse oximetry, for instance, separates oxygenated from deoxygenated hemoglobin by exploiting the additivity of two pure-component spectra at two wavelengths. If absorbances were not additive, the entire bedside-monitor industry would be different. On this simulation, additivity held to better than 1% in the worst spot of the spectrum, which roughly matches what calibrated clinical devices report at clinically interesting saturation levels.

## Loose ends

With another week I would run an actual bench experiment, with a real methylene-blue stock, a 1 cm cuvette, and a low-cost USB spectrometer, to see where the linear regime breaks in practice rather than in simulation. I would also fit a global aggregation model (monomer + dimer + trimer) to the apparent-absorbance curve and back out K values to compare against the published 1960s and 1970s values. The simulation says the calibrated linear regime ends near 10 uM for methylene blue at 664 nm. Anyone with a spectrometer can verify that in an afternoon.
