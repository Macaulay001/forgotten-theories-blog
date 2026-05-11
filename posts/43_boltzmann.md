# I simulated 250 hard disks starting at one speed. Boltzmann's H dropped by 2.3 nats and locked to Maxwell-Boltzmann.

*Part 43 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1872 Ludwig Boltzmann wrote down a single quantity, H = ∫ f ln f dv, and claimed it could only fall. For a dilute gas evolving under his transport equation, dH/dt is non-positive, with equality only when f is the Maxwell-Boltzmann distribution. That was, in effect, the first microscopic derivation of the second law of thermodynamics, and people argued about it for the next half century. Loschmidt said the underlying mechanics is time-reversible so H cannot monotonically fall. Zermelo cited Poincaré recurrence to say it must eventually rise again. Boltzmann shifted to a statistical reading, lost the argument in public, and eventually died by his own hand in 1906. The quantity he defined survived him, mutated into Shannon entropy a generation later, and now sits at the base of half of physics and most of information theory.

I had never actually watched H fall. I had only seen it proved.

## What I tried

The simplest setting where H is non-trivial is a 2D hard-disk gas. N billiard disks in a square box, elastic collisions, energy conserved per pair, no internal degrees of freedom. The hard-disk gas is the toy model Boltzmann's collision operator was written for. Crucially it is event-driven: you can compute the exact time to the next collision in closed form, jump straight there, swap the appropriate components of velocity, and repeat. No timestep, no integration error, no drift.

The plan was to start the system maximally far from equilibrium in the one direction that matters for H. All particles get the same speed v₀ = 1, with random direction. In velocity space they sit on a thin ring of radius 1. The equilibrium target is a 2D Gaussian on (vx, vy), which has a single peak at the origin and zero density on rings. Going from one to the other is what the H-theorem says must happen, and it must happen monotonically (modulo finite-N noise).

I wrote an event-driven simulator from scratch. A min-heap of (time, kind, i, j) events with invalidation counters per particle. At each collision, recompute next events for the two participants only. Wall reflections handled the same way. N = 250 disks, radius 0.012, in a unit box, with packing fraction around 0.11 (well below the freezing density, so the gas is genuinely dilute). Total simulation 20 time units, sampled at 160 snapshot times.

```python
# Elastic disk-disk collision, equal mass.
dr = pos[j] - pos[i]
dv = vel[j] - vel[i]
sigma = 2 * r
J = (dr @ dv) / sigma
jvec = J * dr / sigma
vel[i] = vel[i] + jvec
vel[j] = vel[j] - jvec
```

At each snapshot I binned (vx, vy) into a 40×40 grid and computed H = Σ fᵢⱼ · ln(fᵢⱼ) · ΔA from the density estimate fᵢⱼ. That introduces a binning-dependent offset relative to the analytical integral, which I dealt with at analysis time by computing the same estimator on N samples drawn directly from the Maxwell-Boltzmann target. Comparing the simulator's plateau to MB samples under the same estimator is the only fair test of "did it reach equilibrium".

## What happened

The trace fell hard and locked.

![H(t) for 250 hard disks, monoenergetic start](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/43_h_vs_time.png)

At t = 0 the empirical H is +0.876, because the 250 velocities concentrate near the |v| = 1 ring and the histogram puts most of its mass into the 40 or so annular cells that intersect the ring. By t = 1 the first wave of collisions has scattered enough particles off the ring that H has dropped to roughly -1.0. By t = 5 H is around -1.35. By t = 15 it sits between -1.43 and -1.50, with small fluctuations.

The reference value, computed by drawing 250 fresh samples from a 2D Gaussian with σ² = T = 0.5 and running them through the same histogram estimator, comes out to -1.43, with a sample-to-sample standard deviation near 0.03 over 50 redraws. The simulated plateau, -1.46 averaged over the last 20 snapshots, agrees with the equilibrium target to within one sample-noise unit.

| quantity                        | value      |
|---------------------------------|------------|
| H(0), monoenergetic start       | +0.876     |
| H plateau, mean of last 20      | -1.457     |
| H equilibrium, MB samples       | -1.432     |
| Total drop in H                 | 2.33 nats  |
| <v²> conservation               | 1.0000     |

Two finer points came out of the trace. First, H is not strictly monotone. Between t = 5 and t = 10 it actually rose by about 0.07 nat before falling back. This is the Loschmidt-Zermelo correction made visible. A 250-particle system has H fluctuations of order σ(H) ≈ 0.03 in the equilibrium state, and during the slow approach the fluctuations are larger. Boltzmann's monotone fall is a statement about the ensemble average, not about every realization. With 250 disks you can watch the statistical character of the second law directly.

Second, energy was exactly conserved. <v²> stayed at 1.0000 throughout, which it has to under elastic collisions. The H-theorem and energy conservation pin the equilibrium to the unique distribution that maximizes entropy at fixed energy, which is Maxwell-Boltzmann. Watching the system pick that distribution out of the space of all velocity distributions, with no input other than pairwise elastic scattering, is the closest thing physics has to a derivation of "why" entropy is a thing.

The speed histogram tells the same story in a less abstract way.

![Speed distribution evolution](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/43_speed_evolution.png)

At t = 0 it is a spike at |v| = 1. At t = 5 it has spread into a wide hump that already looks roughly Maxwellian. By t = 20 the histogram tracks the analytical 2D Maxwell speed PDF, (v/T) exp(-v²/2T) with T = 0.5, inside the bin-to-bin noise of 250 samples.

## What it may mean

The H-theorem holds, in the sense that a 2D hard-disk simulator that knows nothing about thermodynamics, only about how to bounce two disks, reproduces Boltzmann's prediction in detail. That was the expected outcome. The interesting parts are the ones a 1872 derivation could not see.

The fluctuations are real and they are not small. At N = 250 the per-snapshot H scatter is about 0.03 nat, and over the relaxation phase you can see the trace wobble upward by twice that. Push N up by a factor of 10 and the wobble shrinks by √10, but never vanishes. Boltzmann eventually wrote that the second law is statistical in exactly this sense. The simulation makes that statement quantitative: at N = 250 the "second law" is good to about 1% in H per snapshot.

The match between simulator and Maxwell-Boltzmann does not just confirm a 19th century formula. It tells you that Boltzmann's *Stosszahlansatz* (the assumption that the velocities of two particles about to collide are statistically independent) is a fine approximation here. That assumption is what makes the H-theorem work in the first place, and it is also what Loschmidt's reversal argument breaks. In a 2D box of 250 dilute disks the correlations are weak enough that molecular chaos is a working approximation across the full 20-unit run.

That may not hold if the gas is dense. The hard-disk system freezes around packing fraction 0.7. Approaching that, the velocity correlations get long-ranged, the H-theorem picks up corrections, and the system stops reaching Maxwell-Boltzmann in any clean way. At packing fraction 0.11 there is no sign of trouble.

## Loose ends

With another week I would rerun the sweep at N = 2000 in a periodic box (no walls), measure the relaxation time as a function of mean free path, and see whether the H trace acquires the predicted exponential approach to the plateau with rate 1/τ_coll. I would also try the harder initial condition of two interpenetrating beams (a bimodal distribution in vx) and watch the joint (vx, vy) density relax. The single-laptop cost for the upgraded sweep is about an hour, and it would let me test the Loschmidt point directly: reverse all velocities at t = 10, watch H climb back up, and measure how long the rise lasts before molecular chaos rerandomizes the system. Boltzmann's answer in 1877 was "a very short time". The simulation should be able to put a number on that.
