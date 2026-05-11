# Wien's 1893 displacement law holds to 1 part in 10,000 across four decades of temperature

*Part 58 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Wilhelm Wien published his displacement law in 1893, seven years
before Max Planck wrote down the full blackbody spectrum. From a
thermodynamic argument about adiabatic compression of cavity
radiation, Wien concluded that the wavelength at which a glowing
body radiates most intensely is inversely proportional to its
absolute temperature:

    lambda_max * T = b

with b a universal constant of nature. He could not say what b was
in modern units. He just argued, from scaling, that the same number
should work for the filament of a lamp, the photosphere of a star,
and the inside of an iron furnace.

That is a strong claim to make seven years before anyone knew what
the spectrum actually looked like. Wien's own attempt at a full
spectral formula (also from 1896) was wrong at long wavelengths, and
it was Planck's interpolation in 1900 that fixed it and kicked off
quantum theory. But the displacement law survived intact, because it
follows from a similarity property of the spectrum that does not
depend on the specific functional form. I wanted to see how well
that holds up on a grid I could actually compute.

## What I tried

The plan was small. Write down Planck's spectral radiance
B(lambda, T) per unit wavelength, evaluate it on a fine log-spaced
grid of wavelengths from 30 nm out to 1 metre, do that for four
temperatures (100 K, 1000 K, 3000 K, 5778 K), and find the argmax.
Then check whether the product lambda_max times T came out to the
same constant for all four.

I picked those temperatures on purpose. 100 K is colder than dry
ice; its peak should sit deep in the far infrared. 1000 K is a
dull-red furnace. 3000 K is an incandescent tungsten lamp. 5778 K
is the IAU's adopted effective temperature for the Sun, so its peak
should land in the visible if Wien is right.

Independently of the grid search, I wanted to confirm the analytic
constant. If you differentiate B(lambda, T) with respect to lambda
and set the derivative to zero, you get the transcendental equation

    x = 5 (1 - e^-x)

with x = h c / (lambda_max k_B T). The non-trivial root sits a bit
under 5. From that root you read off b = h c / (k_B x_W). I solved
the transcendental with scipy's brentq.

The whole computation is a few dozen lines and runs in about a
second on a laptop. The interesting question is not whether the law
holds (it has to, given Planck's formula). It is whether a coarse
grid actually recovers the analytic peak, and how the four
temperatures' products line up.

## What happened

The four peaks landed where Wien said they should.

![Planck curves with peaks marked](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/58_planck_curves.png)

| T (K) | lambda_max | lambda_max * T (mm K) |
|---|---|---|
| 100 | 28.98 micrometres | 2.8978 |
| 1000 | 2.898 micrometres | 2.8979 |
| 3000 | 0.9659 micrometres | 2.8977 |
| 5778 | 0.5015 micrometres (green) | 2.8977 |

The product agrees across four orders of magnitude in temperature.
The numerical scatter, about 0.0001 mm K, is grid-resolution noise,
not physics. Refining the wavelength grid near the peak pushes the
disagreement down further.

The transcendental root came out to x_W = 4.965114232. Plug it into
b = h c / (k_B x_W) and you get 2.89777 x 10^-3 m K, which is the
modern published value to the digits I am quoting. The grid argmax
hits this to five significant figures.

Here is the core of the calculation. No imports, no boilerplate.

```python
def planck_lambda(lam, T):
    a = 2.0 * h * c * c / lam**5
    arg = h * c / (lam * kB * T)
    return a / (np.exp(arg) - 1.0)

x_W = brentq(lambda x: x - 5.0 * (1.0 - np.exp(-x)), 1e-6, 20.0)
b   = h * c / (kB * x_W)
for T in [100.0, 1000.0, 3000.0, 5778.0]:
    B = planck_lambda(lam, T)
    print(T, lam[np.argmax(B)] * T)
```

A separate plot of lambda_max times T against temperature is the
clearest illustration of how flat the relationship is. Four points
sit, indistinguishable to the eye, on a horizontal line at
2.898 mm K.

![Wien product is flat](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/58_wien_product.png)

Then the cosmic microwave background. The CMB has a temperature of
T = 2.7255 K (COBE/FIRAS). Wien predicts a B_lambda peak at
b / T = 1.063 mm. That is the millimetre-wave radio band, which is
why the discovery instruments in the 1960s used horn antennas built
for radar wavelengths.

A subtlety came up here. FIRAS quotes the peak of B_nu, the
frequency-density form of the spectrum, at about 160.23 GHz. If you
naively convert that frequency to a wavelength via c/nu you get
1.87 mm, not 1.06 mm. The two are not contradictory. B_lambda and
B_nu peak at different physical wavelengths because the Jacobian
dnu/dlambda = -c/lambda^2 redistributes the spectral density. Each
form has its own Wien constant, and Wien's displacement law for
frequency uses a different transcendental root (the solution of
x = 3 (1 - e^-x), roughly 2.821). I had to remind myself of this
the second the numbers disagreed.

## What it may mean

Wien's argument is the cleanest example I can think of in physics
of a scaling result outliving the framework that produced it. The
1893 derivation does not need quantum theory. It needs only that the
spectrum, written as a function of lambda and T, has the form
B(lambda, T) = lambda^-5 f(lambda T). That single similarity is
enforced by classical thermodynamics applied to a slowly
compressed cavity. Whatever f turns out to be, the peak must scale
inversely with T. So when Planck fixed the spectrum in 1900, the
displacement law was already locked in.

That is also why every non-contact thermometer works. A pyrometer
trained on a furnace, or a satellite radiometer staring at the
Earth's surface, or a spectrograph measuring the colour temperature
of a star, all read out temperature by locating where the spectrum
peaks (or where the slope changes sign) and inverting Wien's
constant. The same constant that fixes the Sun's green peak at
502 nm also fixes the CMB peak at about a millimetre. One number,
working from cold gas clouds to white-hot tungsten.

The thing to take away may be that good physics often comes from a
scaling argument rather than a specific model. Wien did not have
the right spectrum. He had the right similarity. The model came
later and ratified the scaling.

## Loose ends

The check here is purely numerical, not experimental. A real test
against laboratory blackbody radiation has to deal with emissivity
below 1, atmospheric water and CO2 absorption (which mostly closes
the 5 to 8 micron window), and the specific detector bandpass.
None of those break Wien, but they change what you measure.

The B_lambda / B_nu point is worth flagging. If you ever see "the
peak of the CMB is at X" without specifying which form of the
spectrum, the number is ambiguous by a factor of about 1.76. With
another week I would write a short follow-up that walks through the
three Wien constants (wavelength, frequency, and the entropy peak
at yet a third value) and shows how the Jacobian moves them around.
You can replicate everything here in about thirty seconds on a
laptop.
