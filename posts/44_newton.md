# Newton's 1701 cooling law fits my simulated coffee perfectly, and hides a 26% error in the rate constant

*Part 44 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Isaac Newton published an odd little paper in 1701 called *Scala graduum
Caloris*. Among other things it laid down what generations of physics
students now recite as Newton's law of cooling: a hot object loses
temperature at a rate proportional to how much hotter it is than its
surroundings. Write it out and you get

dT/dt = -k (T - T_amb)

Integrate and the body decays exponentially to room temperature. It is
the first differential equation most students ever solve. It is also
the model behind forensic time-of-death estimates and a lot of food
safety guidance.

Here is what bothered me. Newton wrote that in 1701, before the kinetic
theory of gases, before Fourier's heat equation, and 178 years before
Stefan worked out that radiated power goes as the fourth power of
absolute temperature. So how can a law that ignores radiation entirely
still feel right for a cup of coffee or a forensic cadaver? Either
radiation is small in everyday conditions, or the exponential is doing
some quiet bookkeeping that hides the missing physics. I wanted to know
which.

## What I tried

The setup is small enough to run on a laptop. Take a body at 90 C
sitting in 20 C ambient. Cool it for an hour. Compare three physics:

1. Pure Newton: dT/dt = -k (T - T_amb)
2. Newton plus Stefan-Boltzmann radiation: an extra term proportional
   to (T_K)^4 minus (T_amb_K)^4
3. Natural-convection nonlinearity: dT/dt proportional to (T - T_amb)
   raised to the 5/4 power, which is what textbook heat-transfer
   correlations give for a vertical plate in still air

For the radiative case I picked a coupling strength such that at the
hottest moment (DT = 70 C) the radiative term carries about 30% of
the total heat loss. That is not a wild number for a dark-painted
metal object in still air. Then I added 0.4 C of Gaussian noise on
top, which is roughly what a cheap thermocouple gives you, and asked
the question every undergraduate asks: does Newton's exponential fit
the data?

Curve-fitting was three free parameters: k, T_0, and an effective
ambient temperature. I did not constrain any of them. I let scipy's
curve_fit find whatever values it wanted.

Here is the core of the simulation. The right-hand side of the ODE
when radiation is included looks like this:

```python
def rhs_radiative(T, t):
    dT_lin = T - Tamb
    dT_rad = (T + 273.15)**4 - (Tamb + 273.15)**4
    return -k_lin * dT_lin - k_rad * dT_rad

T_true = odeint(rhs_radiative, 90.0, t).flatten()
T_obs  = T_true + rng.normal(0, 0.4, t.size)
popt, _ = curve_fit(newton_exp, t, T_obs, p0=[1e-3, 90, 20])
```

The fitted parameters then get compared back against the inputs I put
in, and the residuals get binned by temperature difference so I can
see where the model is leaking.

## What happened

The fit looks beautiful. Plot the noisy "observations" on top of the
exponential and you cannot tell anything is wrong by eye.

![Cooling curves and Newton fit](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/44_cooling_fit.png)

Three things came out of the fit:

| Quantity | True input | Fitted |
|---|---|---|
| Linear rate constant k | 1.50 x 10^-3 /s | 1.88 x 10^-3 /s |
| Initial temperature T_0 | 90.0 C | 89.6 C |
| Ambient T_amb | 20.0 C | 20.1 C |

T_0 and T_amb come back almost exactly right. The rate constant comes
back 26% too high. That is the bookkeeping I was worried about. The
exponential cannot represent a (T_K)^4 term, so it absorbs the extra
cooling by inflating k. If you took this fitted k and applied it to a
different temperature range, say cooling from 50 C to 25 C, you would
overshoot the real cooling speed by a meaningful amount.

The residual plot tells the cleaner story. I ran the fit on the
noise-free radiative truth so I could see structure, not noise:

![Residuals of Newton fit](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/44_residuals.png)

The residuals are tiny in absolute terms, around 0.09 C RMS, with a
maximum near 0.46 C. But they are not random. There is a clear
S-shaped pattern: at high DT the real curve cools faster than the
exponential, in the middle range the real curve cools slower, and at
low DT the two converge. If you bin the residual by DT and plot it,
the shape jumps out. That S is the signature of a nonlinear loss term
that an exponential cannot fit, only approximate.

With 0.4 C measurement noise, this S-shape is buried. The residual
RMS goes up to 0.41 C, basically the noise floor. You would need a
thermocouple about ten times better to see the deviation, or you
would need many independent cooling runs averaged together. That is
probably why Newton's law has survived three centuries of undergraduate
labs untouched. The signal is below the noise of every reasonable
thermometer in every reasonable classroom.

The natural-convection variant (DT^{5/4}) shows a similar but milder
distortion. Same direction, same S-shape, smaller amplitude. So both
of the realistic physical corrections leak into k in the same way:
they make the fitted rate look faster than the true linear coupling.

A practical consequence. If you measure a cadaver's temperature at
two moments and fit Newton's law to back-extrapolate the time of
death, your k is calibrated by whatever range of DT happens to be
between your two readings. Extrapolating that k either earlier (when
the body was hotter and radiation mattered more) or later (when it is
near ambient) will be systematically biased. Forensic pathologists
already know this, which is why the Henssge nomogram and Marshall &
Hoare's two-exponential model exist. But the bias is invisible if
you only fit within a narrow window.

## What it may mean

Newton's law is not wrong. It is one term of a Taylor expansion of the
true heat-loss equation around DT = 0. For small DT it dominates. For
larger DT the radiative and convective terms add corrections that the
linear model cannot represent, and a least-squares fit responds by
shifting its parameters to swallow the difference.

The interesting part is how invisible this is in practice. With
realistic instrument noise, the residual structure disappears. The fit
quality looks excellent. Only by simulating with no noise can you see
that the model is the wrong shape, and only by comparing the fitted k
to a known input can you measure the bias.

I suspect this pattern recurs across physics. Simple laws survive not
because they are exactly right, but because they have enough free
parameters to absorb their own omissions, at least over the range
where they were calibrated. In this sample, that absorption costs
about 26% of the rate constant. Whether that matters depends on
whether you are using k for back-extrapolation, design, or just
classroom demonstration.

## Loose ends

The radiative coefficient I used is a plausible guess, not a measured
value. A real test would replicate this with an actual cooling object,
ideally one painted to give a known emissivity, and a thermometer
better than 0.05 C. I did not include evaporative cooling, which
matters for liquids and skin and which adds its own nonlinear
temperature dependence. The exponential's tolerance for being wrong is
the headline, and that finding holds regardless of the exact
coefficient. With another week I would dig out the Marshall and Hoare
1962 cadaver data and refit it with explicit radiation, and see
whether the published two-exponential really is two cooling channels
or just one fit absorbing a fourth-power term.
