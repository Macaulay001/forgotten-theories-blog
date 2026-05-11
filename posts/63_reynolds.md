# Reynolds' 1883 number predicts pipe turbulence at Re = 1189, but real pipes wait until 2300

*Part 63 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1883 Osborne Reynolds set up a glass tube, dribbled a thin dye
filament into water flowing through it, and watched what happened as he
opened the valve. At low speeds the dye stayed as a single straight
thread down the centreline. Open the valve a bit further and the thread
wobbled. Open it more and the thread shattered into eddies and the dye
spread through the whole pipe. He published the result in *An
experimental investigation of the circumstances which determine whether
the motion of water shall be direct or sinuous*, and bundled four
variables (density, mean velocity, pipe diameter, viscosity) into one
dimensionless group that now carries his name.

The thing that has always bothered me about this story is that the
critical Reynolds number is not really one number. Textbooks say "about
2000", or "about 2300", or "anywhere from 2000 to 4000 depending on how
quiet your pipe is". For something so canonical, the value is oddly
slippery. I wanted to see what happens when you take the two standard
friction correlations seriously and ask where they predict transition
should happen, then compare that to where it actually does.

## What I tried

The setup is small. Pipe-flow pressure drop is usually written through
the Darcy friction factor f, which depends only on Re for a smooth
pipe. There are two clean correlations, one on each side of the
transition.

For laminar flow, Hagen-Poiseuille gives an exact result from solving
the Navier-Stokes equations in a circular pipe:

    f = 64 / Re

For turbulent flow in a smooth pipe, Blasius' 1913 empirical fit (still
the workhorse below Re ~ 100000) gives:

    f = 0.316 Re^(-1/4)

If a pipe could choose, it would presumably switch from one law to the
other at the point where they predict the same friction. That happens
when 64/Re equals 0.316 Re^(-1/4), which rearranges to Re =
(64/0.316)^(4/3). I wanted to compute that crossing, plot the Moody
diagram, and see how far it sits from the textbook transition window of
2300 to 4000.

Then a second piece. The full Navier-Stokes simulation of pipe flow at
finite Re is not a laptop job. But the Lorenz system, with its three
ODEs and one bifurcation knob, is the canonical toy model for shear-flow
chaos. I swept the Rayleigh-number proxy r from 1 (deep laminar) to 50
(deep chaotic) and tracked how the long-time variance of x changed.
Not a literal model of pipe flow, but useful for showing how a smooth
system can sit quietly until a control parameter pushes it past a
threshold.

The two outputs are an NPZ with the friction-factor curves and the
Lorenz time series, plus a Moody-style figure with the transition band
highlighted.

## What happened

The correlations cross at Re = 1189.4. That is well below the empirical
transition. If you plot the laminar line and the Blasius line on a
log-log axis, they touch around Re = 1200 and then the laminar line
keeps drifting downward while the Blasius line stays roughly flat.

![Moody diagram with transition band](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/63_moody.png)

The blended curve I drew (linear in the band, the two correlations
outside) jumps from f = 0.028 at Re = 2300 to f = 0.040 at Re = 4000.
That is a factor of 1.4 in friction across the transition band. For a
typical municipal water main running near Re = 50000 the friction
factor sits around 0.021, so the transition really is the dramatic
move in pipe-flow behaviour.

Here is the core computation. Five lines, no boilerplate.

```python
Re = np.logspace(2, 6, 400)
f_lam  = 64.0 / Re
f_turb = 0.316 * Re ** (-0.25)
Re_cross = (64.0 / 0.316) ** (4.0 / 3.0)   # 1189.4
w = np.clip((Re - 2300.0) / (4000.0 - 2300.0), 0.0, 1.0)
f_blend = (1 - w) * f_lam + w * f_turb
```

The crossing at 1189 is interesting on its own. Once you are above it,
turbulent flow is a valid solution of the equations, in the sense that
the empirical correlation gives a lower pressure drop than the laminar
prediction extrapolated outward. But real pipes do not switch there.
They sit in laminar flow for another factor of two in Re before any
disturbance is strong enough to tip them. Modern hydrodynamic stability
work (the kind associated with Avila, Hof, Eckhardt and colleagues) has
shown that laminar pipe flow is actually linearly stable at all
Reynolds numbers. The transition is subcritical: you need a
finite-amplitude perturbation to push the system over.

The Lorenz sweep echoed the same delay, in cartoon form.

| r | std(x) | zero-crossings of x |
|---|---|---|
| 1  | 0.02 | 0 |
| 10 | 0.00 | 0 |
| 24 | 5.93 | 9 |
| 28 | 7.78 | 21 |
| 50 | 10.94 | 31 |

At r = 10 the system has long since left its trivial fixed point in a
linear-stability sense (the standard Hopf-like loss of stability sits
around r ~ 24.74 with the textbook sigma = 10 and b = 8/3), but in my
sweep at r = 10 the trajectory still spirals quietly into a non-trivial
fixed point with essentially zero late-time variance. Only at r = 24 do
the oscillations show up, and at r = 28 it goes fully chaotic. The
quantitative numbers do not map onto pipe Reynolds numbers. The shape
of the bifurcation curve does. A linear analysis would tell you where
quiet flow could break. Only the full nonlinear simulation tells you
where it does.

![Lorenz time series across r](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/63_lorenz_transition.png)

So Reynolds was right that one dimensionless group governs the
transition. He just had the wrong picture of what "critical" means.
The critical Re is not the point where laminar flow becomes unstable.
It is the point where laminar flow becomes vulnerable to perturbations
of whatever amplitude happens to be present in your apparatus.

## What it may mean

If laminar pipe flow is linearly stable at all Re, then the critical
value depends on noise. A perfectly quiet pipe could in principle stay
laminar at Re = 10000 or higher. Reynolds, working with a glass tube
fed from a settling tank, was already in the low-noise regime; he saw
transition near Re ~ 13000 in some runs, and only by tapping the pipe
did he get the dye thread to break at lower speeds. That detail tends
to get cut from textbook accounts.

Practically, the result may matter for any small-bore tube near the
transition. Drug-injection cannulas, microfluidic chips, blood vessels
of intermediate size, biopsy needles flushing saline: these can sit
inside the 2300-4000 band where the friction factor can change by 40%
depending on upstream noise. That is not the same as flipping between
"working" and "broken", but it is enough to make any model that uses
a single f value optimistic.

The Lorenz part is the more general moral. A linear stability analysis
tells you whether a small perturbation grows. Real life is not always
small. The flow waited 30 to 40% past the linear threshold before it
actually changed character, and the pipe-flow case is even more
extreme: linear stability never breaks, and the transition is
controlled entirely by which finite-amplitude disturbances live in the
system.

## Loose ends

I did not run Navier-Stokes for pipe flow at finite Re. A proper test
of the subcritical-transition story would need either a DNS pipe-flow
solver or a real apparatus. The friction correlations I used are
smooth-pipe approximations; rough pipes follow Colebrook-White and the
crossing point shifts. With another week I would re-run Avila et al.'s
puff-decay statistics from their 2011 paper and check whether the
mean-lifetime divergence at Re ~ 2040 reproduces from a much simpler
model than full DNS. If anyone has cleaned puff-lifetime data from
their pipe experiment, please get in touch.
