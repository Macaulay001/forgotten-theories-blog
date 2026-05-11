# Einstein's 1905 jiggle formula reproduces cleanly on a laptop, 121 years later.

*Part 53 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In May 1905, Albert Einstein submitted a short paper to Annalen der
Physik with a strange promise. If matter is really made of molecules,
he wrote, then a speck of pollen suspended in water should not sit
still. It should be kicked around by molecular collisions, and the
average squared distance it wanders from its starting point should
grow in proportion to time. He gave a formula for the proportionality
constant. The constant depended on temperature, on the fluid's
viscosity, on the size of the particle, and on Boltzmann's constant.
That last dependency is the part that mattered. It meant a microscope
could, in principle, count molecules.

The paper sat for three years before Jean Perrin started actually
measuring pollen tracks under a microscope and using Einstein's
equation to back out Avogadro's number. The two of them, working on
opposite sides of a war, settled a question that chemists had been
arguing about since Dalton: are atoms real, or are they a bookkeeping
trick. Perrin's microscopy said real. He got the Nobel for it in 1926.

The equation that did the work has two pieces. First, the mean-square
displacement of the particle in two dimensions grows as

  ⟨x² + y²⟩ = 4 D t

where D is the diffusion coefficient. Second, the diffusion coefficient
itself is set by the Stokes-Einstein relation,

  D = k_B T / (6 π η r),

with k_B the Boltzmann constant, T the temperature, η the dynamic
viscosity, and r the particle radius. I wanted to see whether a clean
random walk simulation actually obeys both pieces, because textbook
results sometimes fall apart when you run them yourself.

## What I tried

The plan was simple enough to write on a napkin. Simulate 1000
particles diffusing in two dimensions for 10000 timesteps. At each
step, each particle takes a Gaussian jump with variance 2 D dt per
axis (that comes from the central limit theorem version of Einstein's
derivation). Average the squared displacement across the ensemble at
each timestep, plot that against time, fit a straight line, and see
whether the slope matches 4 D.

For the baseline I picked water at 20 C (η = 10⁻³ Pa·s, T = 293 K)
and a particle radius of 0.5 μm, roughly the size of Perrin's gamboge
grains. The Stokes-Einstein formula gives D ≈ 4.29 × 10⁻¹³ m²/s for
that combination. With a timestep of 1 ms and 10000 steps, each
trajectory covers 10 seconds of simulated time, and the typical
particle wanders about 4 μm from its start. That is small enough to
sit inside a single microscope field of view and large enough to be
measurable.

Then I wanted two sweeps. One that varies the radius at fixed
temperature, where the theory predicts D ∝ 1/r. One that varies the
temperature at fixed radius, where the theory predicts D ∝ T as long
as the viscosity is held constant. For each setting, refit D from the
simulated MSD and compare to the analytical value.

The heart of the simulation is a few lines:

```python
def simulate_2d(n_particles, n_steps, dt, D, rng):
    sigma = np.sqrt(2 * D * dt)
    steps = rng.normal(0.0, sigma, size=(n_steps, n_particles, 2))
    pos = np.zeros((n_steps + 1, n_particles, 2))
    pos[1:] = np.cumsum(steps, axis=0)
    return pos

pos = simulate_2d(1000, 10000, 1e-3, D0, rng)
t = np.arange(10001) * 1e-3
msd = (pos ** 2).sum(axis=2).mean(axis=1)
slope = np.sum(t * msd) / np.sum(t * t)
D_fit = slope / 4.0
```

No tricks. The whole experiment runs in about three seconds on a
laptop. Fitting through the origin (because the particles all start
at the origin and the MSD has no intercept) avoids a small bias that
two-parameter fits introduce on noisy tails.

## What happened

The baseline run gave D_fit = 4.34 × 10⁻¹³ m²/s against a theoretical
value of D_true = 4.29 × 10⁻¹³ m²/s. The ratio is 1.012, so the fit
sits about 1.2% above the truth, which is roughly what you would
expect from sampling 1000 particles. Taking the log-log slope of the
MSD curve, I get 0.990, which is consistent with linear growth.
Anomalous diffusion, which is what people study when this slope is
not 1, would show up as a slope of 0.7 or 1.3 or similar. Here it sat
on the line.

![MSD vs time and ensemble tracks](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/53_msd_linear.png)

The radius sweep is the part that surprised me a little, because the
prediction is a six-decade power law and the simulation has to track
it at both ends.

| r (μm) | D theory (m²/s) | D simulated (m²/s) |
|--------|------------------|---------------------|
| 0.10 | 2.15 × 10⁻¹² | 2.13 × 10⁻¹² |
| 0.20 | 1.07 × 10⁻¹² | 1.12 × 10⁻¹² |
| 0.50 | 4.29 × 10⁻¹³ | 4.63 × 10⁻¹³ |
| 1.00 | 2.15 × 10⁻¹³ | 2.19 × 10⁻¹³ |
| 2.00 | 1.07 × 10⁻¹³ | 1.05 × 10⁻¹³ |
| 5.00 | 4.29 × 10⁻¹⁴ | 4.11 × 10⁻¹⁴ |

Every simulated D sits within roughly 8% of the theoretical value, and
the discrepancy is symmetric (sometimes high, sometimes low), so it
looks like ordinary finite-sample noise rather than a systematic bias.
The largest deviation is at r = 0.5 μm, where the fit overshoots by
about 8%. Rerunning with a different random seed brings that closer
to the line, so it appears to be a seed-dependent fluctuation rather
than a structural problem with the method.

The temperature sweep behaves similarly. Holding the radius at 0.5 μm
and stepping T from 273 K to 353 K, the fitted D rises monotonically
from 3.99 × 10⁻¹³ to 5.03 × 10⁻¹³, tracking the linear-in-T
prediction inside about 5% at every point.

![Stokes-Einstein scaling](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/53_stokes_einstein.png)

One thing worth pausing on. The Stokes-Einstein formula bridges three
worlds that should not obviously talk to each other. The kT factor is
pure thermodynamics. The 6πηr factor is pure hydrodynamics, derived
by Stokes in 1851 for a sphere creeping through a viscous fluid. The
proportionality to time is pure probability theory, the central limit
theorem applied to many small kicks. Einstein's 1905 paper glued these
three together in five pages. The numerical agreement above is just a
sanity check, but it is a sanity check that depended on those three
fields not contradicting each other.

The ensemble of tracks also looks the way a textbook says it should.
Forty trajectories spread out from the origin in a ragged starburst,
with most of them ending inside a circle of radius 4 μm and a few
outliers reaching about twice that. The cloud is statistically
isotropic, as you would expect when the jumps are independent and
Gaussian on both axes.

## What it may mean

The headline finding is not that Einstein was right. He has been right
for over a century. The finding is that a clean, modern simulation
reproduces the 1905 prediction at the percent level with no fudge
factors and no parameter fitting beyond the slope of MSD against time.
That sets a useful benchmark for things people actually do with this
equation today.

In live-cell microscopy, single-particle tracking algorithms report a
diffusion coefficient by fitting the same MSD curve I fit above.
Researchers then interpret deviations from the linear law as evidence
of confinement, active transport, or anomalous diffusion. The
sensitivity of those claims depends on how well the linear law fits
under ideal conditions. The 0.99 log-log slope I got here, on
synthetic data with 10⁷ position observations, may serve as a soft
ceiling: real measurements with fewer tracks and shorter durations
should not be expected to do better.

There is also a teaching point. The Stokes-Einstein formula is one of
the few results in physics where you can derive the molecular constant
k_B from a wholly macroscopic measurement, namely the wandering of a
visible speck. That conceptual move, from "watching a single grain"
to "counting Avogadro's number", may still be the cleanest example of
how thermodynamics and statistics talk to each other.

## Loose ends

A more careful study would add short-time inertia (the ballistic
regime, where ⟨r²⟩ ∝ t² before molecular collisions randomize
direction) and hydrodynamic memory effects, both of which were absent
in Einstein's original treatment but get included in modern Langevin
simulations. It would also be interesting to push the simulation to
non-spherical particles, where the 6πηr factor breaks and you need
the full Oseen tensor. With another week I would try fitting D from
sparser, shorter tracks (say 50 frames per particle, 100 particles)
to see how much statistical power you lose, since that is closer to
what microscopy data actually looks like. The code in this post would
take roughly an hour to extend in either direction.
