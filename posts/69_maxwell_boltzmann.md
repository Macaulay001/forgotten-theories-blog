# I drew 5000 nitrogen molecules from Maxwell's 1859 formula. The mean speed landed at 477.9 m/s against a theoretical 476.2.

*Part 69 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In February 1859, two months after publishing the first treatment of Saturn's rings, James Clerk Maxwell wrote down a guess. The speeds of molecules in an equilibrated gas, he proposed, follow a specific functional form: f(v) is proportional to v² times exp(-m v² / 2kT). It came from one assumption, that the three Cartesian velocity components are independent Gaussians with the same variance, plus the requirement that the joint density depend only on the magnitude of the velocity. From those two lines you get a curve that peaks at v_mp = √(2kT/m), has mean ⟨v⟩ = √(8kT/πm), and rms v_rms = √(3kT/m). The three speeds are not equal. Their ratios are pure numbers: 1, 1.1284, 1.2247.

Maxwell did not measure them. He could not. The kinetic theory of gases was barely a theory at that point, and the experimental tools to weigh nitrogen molecules by speed (effusion through pinholes, atomic beams, time-of-flight) all came later. The formula sat on paper for thirteen years until Boltzmann picked it up in 1872 and showed that it is the unique fixed point of the collisional transport equation, the place where his H-functional bottoms out. After that the formula became indispensable. Half of physical chemistry presupposes it.

I wanted to watch it self-assemble. Not derive it. Watch it.

## What I tried

Two pieces.

The first piece is direct sampling. If Maxwell's claim is that the components are independent N(0, kT/m), then drawing 5000 vectors with those components and binning their magnitudes ought to reproduce the curve to whatever precision 5000 samples afford. That is one numpy call. The check is whether the empirical mean, the empirical rms, and the empirical CDF land where the closed-form integrals say they should.

I used nitrogen at 300 K because it is the easiest case to picture. N₂ has a per-molecule mass of 4.65 × 10⁻²⁶ kg. Plug into the three formulas and you get v_mp = 422.0 m/s, ⟨v⟩ = 476.2 m/s, v_rms = 516.8 m/s. So a typical nitrogen molecule in this room is moving a bit faster than the speed of sound. Most of them are. A small fraction, the tail above roughly three v_rms, is moving fast enough that if it could escape Earth's gravity it would.

The second piece is convergence from a non-equilibrium start. The kinetic theory says the Maxwell-Boltzmann distribution is not just one possible distribution among many, it is the attractor under elastic pairwise scattering. I wanted to see that. So I initialized 5000 particles at the same speed v₀ = v_rms, with random directions on the unit sphere. In velocity space they sit on a thin spherical shell, the worst possible initial condition for the speed marginal: every particle has identical |v|.

Then I let them collide. Not in a full hard-sphere molecular dynamics sense, which would require a box and a mean free path, but in the Bird DSMC sense. Each step, pick N random pairs, and for each pair do an elastic exchange that preserves the pair's center-of-mass velocity and the magnitude of its relative velocity while randomizing the direction of the relative velocity. That is exactly the collision operator the H-theorem assumes, stripped of spatial structure. The question is what it does to the speed marginal over time.

```python
# DSMC pair-elastic update for one collision.
vi, vj = vel[i], vel[j]
vcom  = 0.5 * (vi + vj)
vrel  = vi - vj
mag   = np.linalg.norm(vrel)
d     = rng.normal(size=3)
d    /= np.linalg.norm(d)         # uniform on the sphere
new_rel = mag * d
vel[i]  = vcom + 0.5 * new_rel
vel[j]  = vcom - 0.5 * new_rel
```

Energy is conserved exactly per pair (the relative magnitude is preserved), so the total kinetic energy of the gas is a constant of the simulation. Momentum is conserved exactly per pair as well. The only thing the update changes is the *direction* of the pair's relative velocity, and the cumulative effect of randomizing many such directions is what relaxes the speed distribution.

## What happened

The direct draw matched.

![Maxwell-Boltzmann histogram vs analytical PDF](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/69_pdf.png)

The 5000-sample histogram tracks the analytical PDF inside the bin-to-bin Poisson noise across the full range from zero to about 1300 m/s. The vertical dashed lines mark v_mp, ⟨v⟩, and v_rms in their theoretical positions. The histogram does what it should at each of them: zero density at v = 0, a peak around v_mp, and a long tail past v_rms.

Empirically:

| quantity   | empirical | theory  | rel. error |
|------------|----------:|--------:|-----------:|
| ⟨v⟩ (m/s)  | 477.88    | 476.17  | 0.36 %     |
| v_rms (m/s)| 518.94    | 516.84  | 0.41 %     |

A Kolmogorov-Smirnov test against the analytical CDF returns D = 0.011 with a p-value of 0.63. That is what "consistent with the model" looks like in this regime. The KS distance scales as 1/√N for samples drawn from the actual target, and 0.011 at N = 5000 is exactly in the expected band.

One subtlety. The empirical v_mp pulled from an 80-bin histogram comes out at 455 m/s, biased about 30 m/s above the analytical 422 m/s. That looked like a discrepancy at first. It is not. It is the bin width: the mode of a coarse histogram of v² exp(-...) sits one bin to the right of the true peak because the PDF is rising fast on the left of v_mp and falling slowly on the right, so the bin straddling v_mp catches less of the rising flank than the next bin catches of the slower decay. A kernel density estimate or a parabolic fit to the top three bins recovers v_mp to better than one percent. Useful reminder that the mode of a noisy histogram is the least reliable of the three characteristic speeds.

The relaxation from the monoenergetic shell told the same story in motion.

![Relaxation from monoenergetic shell](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/69_relaxation.png)

At step 0 the speed histogram is a spike at v_rms. Every particle has the same |v|, by construction. After a single collision step (which on average gives each particle two collisions), the spike has already broadened into a wide hump. By step 5 the histogram is unimodal and visually Maxwellian, with the peak in roughly the right place. By step 10 it overlays the analytical PDF inside the noise. By step 50 the system sits on the equilibrium curve and stays there.

The mean speed reported in each panel converges toward ⟨v⟩_theory ≈ 476 m/s from its starting value of v_rms ≈ 517 m/s, which makes sense: starting on the shell the mean speed equals v_rms, and the relaxation lowers it to the equilibrium ⟨v⟩, with the difference (about 8 %) absorbed by particles that scatter into the high-speed tail.

Total kinetic energy across the 50-step run was conserved to a relative error of 2.2 × 10⁻¹⁴, which is machine precision. That is by design (the pair update is energy-exact), but it is worth checking, because the H-theorem only forces convergence *to the constant-energy maximum-entropy distribution*. If energy drifts, the equilibrium target drifts with it.

## What it may mean

The functional form is correct, the convergence happens, and the convergence happens on a few-collisions-per-particle timescale. That last number is the one a 1859 calculation could not produce. It says that in a real gas, where each molecule undergoes roughly 10⁹ collisions per second at atmospheric density, the equilibrium speed distribution is established essentially instantaneously on any human timescale. There is no transient. Whatever Maxwell-Boltzmann demands, you get.

This is what makes the formula safe to assume at the base of so many other arguments. The Arrhenius prefactor in chemical kinetics, the effusion rate in mass spectrometry, the thermal velocity in plasma physics, all start by writing down f(v) without comment. The simulation suggests that "without comment" is a defensible move at any density where pair collisions are the dominant interaction, which is to say almost everywhere outside dense liquids and quantum-degenerate regimes.

The piece I cannot rule out is the tail. The high-speed tail above 3 v_rms in a 5000-particle run is sampled by fewer than 50 molecules, and the histogram there is noisy. The Maxwell-Boltzmann form is famous for having a sharper tail than the real distribution does in some non-equilibrium settings (laser-produced plasmas, sub-glass-transition liquids, hypersonic boundary layers). I cannot see that here. With N = 5000 the tail noise swamps any small deviation from exp(-m v²/2kT).

## Loose ends

With another week I would push the sample size to N = 10⁵ and bin the tail above 3 v_rms with care, looking for any sign of kappa-distribution behavior (a power-law correction to the Gaussian tail). I would also rerun the relaxation from a more pathological start, two interpenetrating beams in vx, and watch the joint (vx, vy, vz) density relax. The single-laptop cost is under an hour. The interesting open question, not really mine to settle, is when Maxwell-Boltzmann fails. In which gases, at which densities, in which transient regimes, does the speed distribution acquire a measurable non-Gaussian correction? Aerospace plasma people have been asking that for decades.
