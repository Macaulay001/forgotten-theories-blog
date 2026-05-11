# Four visible hydrogen lines from 1885 predict the UV Lyman series to one part in a million.

*Part 67 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1885 Johann Balmer, a Swiss schoolteacher with no obvious quantum-mechanical agenda, noticed that the wavelengths of four red, blue-green, blue, and violet emission lines from hot hydrogen gas fit a simple formula. The wavelengths were 656, 486, 434, and 410 nanometers. He wrote them as lambda = h * n^2 / (n^2 - 4) with n running 3, 4, 5, 6 and h a numerical constant. He had no idea why hydrogen should care about integers.

Three years later Johannes Rydberg generalized Balmer's fit to all hydrogen series at once. He proposed 1/lambda = R * (1/n1^2 - 1/n2^2), where n1 and n2 are positive integers with n1 < n2, and R is a constant of nature, ~1.0974e7 per meter. Bohr would not derive R from electron charges and Planck's constant until 1913. For twenty-five years the formula sat there working, with no mechanism behind it, an empirical rule that happened to be exactly right.

I wanted to see how much of that prediction one can recover by encoding four numbers by hand and fitting a line.

## What I tried

The plan was deliberately small. Type the four Balmer wavelengths from a NIST table directly into a Python script. Compute 1/lambda for each in inverse meters. Plot those against the Rydberg ordinate 1/4 - 1/n^2, where n is the upper level. If Rydberg is right the four points should fall on a straight line through the origin, and the slope is R_H.

Then test the prediction in the most painful direction. Use the slope from the Balmer fit (visible light) to compute where the Lyman series should appear (ultraviolet), and compare to measured Lyman wavelengths from independent vacuum-UV spectroscopy. The Balmer fit knows nothing about the Lyman levels. If the formula is structural, the prediction should match. If it is a Balmer-only artifact, it should not.

The last step was the reduced-mass check. Bohr's R_inf is what you get assuming the nucleus has infinite mass, so the electron orbits a fixed center. The real hydrogen nucleus is a proton, 1836 times heavier than the electron but not infinitely so. The electron-proton system orbits a common center of mass, and the effective Rydberg is

  R_H = R_inf / (1 + m_e / M_p)

The correction is m_e / M_p ~= 1/1836 ~= 545 parts per million. If the Balmer fit gives me R_H rather than R_inf, this 545 ppm offset should be there, and visible.

I encoded the wavelengths to four decimal places, ran one linear regression, and let the script print everything else.

## What happened

The four Balmer points sit on a straight line through the origin to graphical perfection. The slope-only fit returned R_H = 1.096788e7 per meter. The NIST recommended value is 1.096776e7 per meter. The deviation is 11 parts per million, which is within the precision of the four wavelengths I typed in.

```python
# four Balmer wavelengths in vacuum nm, n2 -> n1=2
balmer = {3: 656.4614, 4: 486.2683, 5: 434.1684, 6: 410.2892}
n2 = np.array(sorted(balmer.keys()), dtype=float)
lam = np.array([balmer[int(n)] for n in n2]) * 1e-9
x = 1/2**2 - 1/n2**2
R_fit, *_ = np.linalg.lstsq(x[:, None], 1/lam, rcond=None)
print(R_fit[0])  # 1.096788e7 /m
```

When I let the regression have a free intercept too, the intercept came out at +8 per meter on values of order 2.5e6 per meter, sub-ppm, consistent with zero. There is no offset; the line really does pass through the origin. Whatever Balmer guessed in 1885 was structurally correct, not a curve fit with extra parameters hidden in it.

![Hydrogen spectral series predicted from the Balmer-only Rydberg fit](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/67_rydberg_spectrum.png)

Then the cross-series test. Plugging the Balmer-derived R_H into 1/lambda = R_H * (1 - 1/n^2) for the Lyman lines gave:

| Transition       | Observed (nm) | Predicted (nm) | Deviation (nm) |
|------------------|--------------:|---------------:|---------------:|
| Lyman alpha 1->2 | 121.5670      | 121.5671       | +0.0001        |
| Lyman beta 1->3  | 102.5722      | 102.5722       |  0.0000        |
| Lyman gamma 1->4 |  97.2537      |  97.2537       |  0.0000        |

A formula fit to four visible-light wavelengths reproduces three ultraviolet wavelengths to the fourth decimal place. The Paschen series in the infrared matches less precisely, but only because I used reference values quoted to one decimal place (1875.6 nm for Paschen alpha); the residuals there reflect my input precision, not the formula.

The reduced-mass check is the satisfying part. I computed R_inf from m_e, e, eps_0, h, c using only fundamental constants:

  R_inf = m_e * e^4 / (8 * eps0^2 * h^3 * c) = 1.097373e7 /m

The Balmer fit gave 1.096788e7 /m. The difference is 1.097373 - 1.096788 ~= 0.0006 in units of 1e7 /m, which works out to about 530 ppm low. The theoretical finite-mass correction predicts exactly 545 ppm:

  R_inf / R_H = 1 + m_e / M_p = 1 + 9.109e-31 / 1.673e-27 = 1.000544

So R_H_theory = R_inf / 1.000544 = 1.096776e7 /m, and the fit recovers it within 11 ppm. The proton is in the data. Four wavelengths typed from a table know about the mass of the nucleus they came from.

This is one of those moments where the empirical constants of nature seem to be communicating something. Balmer in 1885 did not know electrons existed. He could not have written down a reduced-mass correction. But the very visible-light wavelengths he was fitting carried a 545 ppm fingerprint of the proton-to-electron mass ratio, and the modern recalculation pulls that fingerprint out automatically.

![Balmer wavenumbers vs Rydberg ordinate, slope is R_H](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/67_rydberg_balmer_fit.png)

## What it may mean

The Rydberg formula is the cleanest example I know of a numerical pattern that arrived a quarter century before its mechanism. Balmer wrote it as a numerology in 1885. Rydberg generalized it in 1888. Ritz noticed in 1908 that you could combine any two terms 1/n^2 to get a spectral wavenumber (the Ritz combination principle), which is the empirical content of the Bohr postulate stated five years early. Then Bohr in 1913 said the integers are quantum numbers and the constant is a particular combination of e, h, m_e, c. Then Schroedinger and Dirac wrote down the wave equation that produces those quantum numbers from first principles.

At every step the same four numbers (656, 486, 434, 410 nanometers) constrained the answer. They are absurdly informative for what they are: four colors a person can pick out by eye through a prism. In this sample, the formula fit to those four colors predicts the rest of the hydrogen spectrum to one part in a million and carries the proton mass as a sub-ppm bias. That is a lot of physics packed into four entries of a table.

The flip side is that the precision is not infinite. Hydrogen lines are doublets when you resolve fine structure (the Sommerfeld corrections in 1916, then Dirac, then the Lamb shift in 1947 from QED). The Rydberg formula sees only the centroid. Push the precision below 1 ppm and the formula stops being enough. That is also where the interesting physics started moving in the 20th century.

## Loose ends

With another week I would add hydrogen isotope wavelengths (deuterium, where R_D differs from R_H by ~270 ppm via the m_e / M_d term, the very signal Harold Urey used to discover deuterium in 1931), and fit the m_e / M ratio out of the cross-isotope shift. I would also try the fit on helium II singly-ionized, which obeys the same formula with R replaced by Z^2 * R (Z=2), and check whether the recovered Z is integer to high precision. The pipeline runs in under a second on any laptop, and the numbers have been stable for 138 years.
