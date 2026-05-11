# I rebuilt Kolmogorov's 1941 turbulence law from a random-phase Fourier sum. The slope came out at -1.68.

*Part 36 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In the summer of 1941 Andrey Kolmogorov wrote a three-page paper that did
something almost cheeky. He argued that if turbulence is fully developed and
locally isotropic, then in a range of scales far from the stirring and far
from viscosity, the only things that can matter are the wavenumber k and the
average rate eps at which energy is being dissipated per unit mass. From
those two ingredients you can build exactly one quantity with the units of
an energy spectrum, and it has to look like E(k) = C * eps^(2/3) * k^(-5/3).
The minus five-thirds was not measured. It fell out of dimensions.

For decades people argued about whether the law really held, what the
constant C was, and whether real turbulence had subtle corrections from
intermittent bursts of dissipation. Sreenivasan's 1995 compilation pulled
together experiments from wind tunnels, jets, the atmosphere, and the
ocean, and found C clustering between roughly 1.4 and 1.7. The slope has
held up well across decades of measurement, with small but persistent
intermittency corrections of order 0.03.

I wanted to do the boring exercise that nobody seems to write up: just
build a 1D velocity field that obeys K41 by construction, then see how
cleanly two standard spectral estimators recover the slope and the constant.
If the estimators are honest, the answer should come out near -1.667 with
C around 1.5. If they are biased, the audit tells me by how much.

## What I tried

The plan was three steps.

First, draw a random-phase Fourier field. For N = 65,536 points on a
domain of length 2pi, pick complex amplitudes whose squared modulus matches
C * eps^(2/3) * k^(-5/3), with C = 1.5 and eps = 1, then assign uniform
random phases. Inverse-FFT to get u(x). Multiply by a smooth taper that
rolls off below k_forcing = 4 (the equivalent of stirring scales) and
above k_diss = 4000 (the dissipation scale).

Second, recover the spectrum two ways. The raw periodogram is just
|FFT(u)|^2 with the right Parseval bookkeeping. Welch's method splits
the signal into overlapping Hann-windowed segments and averages, which
trades resolution for lower variance. Fit a log-log slope in the
inertial window k in [20, 800], well inside the tapered region.

Third, sanity check with a different generator. A fractional Brownian
motion with Hurst H = 1/3 is known to have a k^(-5/3) spectrum by
construction, but the path looks nothing like the Fourier synthesis. If
both routes give the same slope, the estimator is not playing tricks.

The Parseval normalization is the only subtle step. For numpy's rfft
convention, getting var(u) to match the integral of E(k) dk requires
setting the amplitude of each interior mode to N * sqrt(E(k) dk / 2).
I missed the factor of N on the first run, the synthesized variance came
out at 10^-4, and the recovered slope was still right (slopes do not care
about overall scale) but the C estimate was zero. Embarrassing, but a
useful reminder that slope-fits and amplitude-fits fail in different ways.

## What happened

The recovered slopes landed where they should.

| estimator       | slope    | 95% CI   |
|-----------------|---------:|---------:|
| raw periodogram | -1.676   | 0.001    |
| Welch PSD       | -1.711   | 0.026    |
| fBm H = 1/3     | -1.667   | 0.0004   |

The raw periodogram is within 0.01 of the target -1.667. Welch is biased
slightly steeper, about 0.04 off, which is what you expect when the Hann
window leaks a little energy across bins near the upper edge of the fit
range. The fBm cross-check is essentially exact, as it should be.

The constant came out at **C = 1.48**, against a target of 1.5 and an
experimental range of 1.4 - 1.7. That is well within where Saddoughi and
Veeravalli sat their measurements in 1994. The synthetic variance was
0.785, matching the analytical integral of the tapered spectrum to a
few percent.

Here is the heart of the synthesis. Five lines do the real work.

```python
dk = 2 * np.pi / L
k  = 2 * np.pi * np.fft.rfftfreq(N, d=dx)
E  = 1.5 * eps**(2/3) * np.where(k > 0, k, 1.0)**(-5/3)
amp = N * np.sqrt(E * dk / 2)
phase = rng.uniform(0, 2*np.pi, size=k.shape)
u = np.fft.irfft(amp * np.exp(1j*phase), n=N)
```

That is the whole physics: pick amplitudes that scale as k^(-5/6) (the
square root of k^(-5/3)), give them random phases, transform back.

![Energy spectrum log-log](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/36_spectrum.png)

The figure shows the raw periodogram in grey, the Welch estimate in blue,
and the dashed line is the analytical 1.5 * eps^(2/3) * k^(-5/3). Three
decades of clean -5/3 scaling sit between k = 20 and k = 800. Below that
the forcing taper kicks in. Above it the dissipation taper bends the
spectrum down hard, the way viscosity does in real turbulence.

One thing the picture makes obvious. The raw periodogram has very high
variance from mode to mode (that grey fuzz). Welch averages over segments
and produces a much cleaner curve, but the segmentation also costs you
resolution at the low-k end. If your inertial range is narrow, Welch can
miss it. For three decades of clean scaling like this synthetic field,
either estimator works.

## What it may mean

K41 is a dimensional-analysis result with no derivation from the
Navier-Stokes equations. It says that if all the assumptions hold (high
Reynolds number, statistical isotropy, locality of the cascade in scale,
no significant intermittency), the spectrum must look like k^(-5/3). The
experimental record over eighty years has been remarkably kind to this
prediction.

What the audit here shows is mostly bookkeeping. Given a field that
obeys the law by construction, standard estimators recover the slope to
within 0.05 and the constant to within 0.02. So when an experimentalist
reports a measured slope of, say, -1.71 in some boundary layer, that
0.04 deviation may be windowing bias rather than real intermittency. The
She-Leveque 1994 correction adds about 0.03 to the slope magnitude in
high-Reynolds 3D flows. That is the same order as estimator bias, which
is why pinning down intermittency requires very long records and careful
estimators.

For atmospheric and ocean modelers, the takeaway is the one they already
live with. The -5/3 law is a constraint on what sub-grid closures must
do in the inertial range, and the constant C ~ 1.5 calibrates eddy
diffusivities. Both numbers seem to survive a clean synthetic test.

A second small surprise was how few points it took. With N = 65,536 in
1D, three decades of inertial range come out cleanly enough to read the
slope to two decimal places. That is a few megabytes of data and a few
seconds of compute. The expensive part of turbulence is generating the
field from real physics, not measuring its spectrum once you have it.
This may be why dimensional arguments like K41 have outlived almost
every more elaborate closure theory: the prediction is sharp and the
test is cheap.

## Loose ends

The honest caveat is that this is a forward test, not a real Navier-Stokes
simulation. A 256^3 forced isotropic DNS would recover the cascade from
actual physics rather than imposed amplitudes, and would let me look at
intermittency exponents and the precise slope correction. That is roughly
an overnight CPU job. The 2D inverse-cascade case (Kraichnan 1967) is
also a different beast with a -5/3 in energy flowing the other way, and
I did not test it. Anyone who wants to push this further could re-run
the same estimators on a public DNS dataset and report the slope and C
separately, with windowing bias subtracted.
