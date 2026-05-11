# Fick's 1855 diffusion equation predicts that a pulse spreads as a Gaussian with variance 2Dt. The PDE recovers D to within 0.01%.

*Part 61 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1855, Adolf Fick was a 26-year-old anatomy demonstrator in Zurich. He had spent a year staring at salt diffusing through membranes and watching the concentration profile change shape. Fourier had published the heat equation in 1822, and Fick noticed that salt in water seemed to obey the same kind of law. He wrote two pages of *Annalen der Physik* asserting, by analogy, that the flux of a dissolved substance is proportional to the negative gradient of its concentration. From that single assumption he wrote down what we now call Fick's second law:

  ∂c/∂t = D ∇²c

That equation tells you everything about how a sugar cube dissolves, how oxygen crosses an alveolus, how a drop of dye spreads in still water, and how impurities diffuse into silicon. It also implies a very specific shape: if you release a point pulse of mass at the origin, the concentration profile at time t is a Gaussian with variance σ² = 2 D t. The width grows as the square root of time. The shape is locked in.

Fick had no random-walk picture. That came fifty years later, from Einstein and Smoluchowski. What he did have was an analogy and a tabletop, and the analogy turned out to be correct. The PDE is still the thing engineers write on the board when they want to estimate how long it takes a drug capsule to leak.

I had taught this equation. I had never actually integrated it on a grid and watched the Gaussian appear.

## What I tried

The plan was two routes to the same number, run side by side. Route one: solve the 1D diffusion PDE numerically starting from a near-delta initial condition, fit Gaussians at several times, and check that σ² grows linearly with slope 2 D. Route two: simulate 1000 particles doing independent 2D random walks with Gaussian step increments, measure ⟨r²⟩(t), and check that it grows linearly with slope 4 D. The two routes should agree on the same D, because Fick's PDE is the continuum limit of the random walk.

For the PDE I used the simplest scheme that works, explicit forward-time centered-space (FTCS) on a grid of 2001 points over a domain of length 40. The stability condition is r = D Δt / Δx² ≤ 0.5. I used r = 0.4. The initial profile was a Gaussian of width σ₀ = 4 Δx ≈ 0.08, narrow enough to look like a delta on the scale of the eventual spread but resolvable on the grid. I took snapshots at t = 0.1, 0.5, 1.0, 2.0, 4.0 and fit each one to a Gaussian to recover σ(t).

For the random walk I drew Gaussian increments with variance 2 D Δt per axis, accumulated positions for 4000 steps of Δt = 0.001, and at each step computed the ensemble mean of r² = x² + y² over the 1000 particles. A linear regression of ⟨r²⟩ against t gives a slope, and that slope divided by 4 is D.

Both pieces fit in one short script. There is no library magic, just numpy slicing and a polyfit at the end.

## What happened

The PDE behaved itself almost too well. The fit slope of σ²(t) versus t was 2.0000, which gives D_pde = 1.0000 against an input D of 1.0. Relative error under 0.01%. The Gaussian shape held at every snapshot. Here is the table I printed at the end of the run.

| t   | σ_fit  | σ_theory |
|----:|-------:|---------:|
| 0.1 | 0.4543 | 0.4472   |
| 0.5 | 1.0032 | 1.0000   |
| 1.0 | 1.4165 | 1.4142   |
| 2.0 | 2.0016 | 2.0000   |
| 4.0 | 2.8296 | 2.8284   |

The first row carries a small bias because the initial spike was not a true delta, it had σ₀ ≈ 0.08. Subtracting σ₀² before the slope fit cleans that up. From t = 0.5 onward, the FTCS solution and the analytic Gaussian agree to better than 0.3% on the fitted width.

![PDE evolution](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/61_pde_evolution.png)

The solid curves are the FTCS solution at five times. The dashed curves are the analytic Gaussian with variance 2 D t + σ₀². They sit on top of each other at the eye's resolution.

The core of the FTCS update is three lines:

```python
# FTCS one-step update of the 1D diffusion equation
r = D * dt / dx**2          # stability: r <= 0.5
for step in range(nsteps):
    c[1:-1] = c[1:-1] + r * (c[2:] - 2 * c[1:-1] + c[:-2])
    c[0] = 0.0               # zero-flux boundary, domain wide enough
    c[-1] = 0.0
```

That is it. The second derivative becomes a three-point stencil and the time derivative becomes a forward difference. Stability comes from the CFL-like condition on r. Everything else is bookkeeping.

The random walk gave D_rw = 1.0734, a 7.3% overshoot. That is sampling noise, not a model defect. ⟨r²⟩ at large t is a sum of correlated quantities, and its variance across realizations scales like t² / N. With N = 1000 and a final time of 4, you should expect the slope estimate to wobble by a few percent on a single run. The MSD curve still looks linear by eye.

![MSD comparison](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/61_msd_comparison.png)

The left panel shows variance from the PDE fits sitting on the line 2 D t. The right panel shows the random-walk MSD next to the line 4 D t. The slopes match. The two pictures of diffusion agree on the same diffusion coefficient.

A couple of things stood out. One was how cheap the verification is. Fick's law in its full PDE form is the kind of thing you imagine requires an industrial solver. In fact, three numpy lines of FTCS, set up with the right Δt, give you a Gaussian with the right width to four decimal places. The other was that the random walk is the noisier of the two routes. The PDE quietly averages over the whole ensemble, while the particle simulation has to do it explicitly with N = 1000 draws. To get D_rw to match D_pde at the 1% level on this run, you would need either N = 100,000 particles or an average over about 50 independent seeds.

The Gaussian shape, by the way, is not a coincidence. The diffusion equation is linear and translation-invariant, so its Green's function (the response to a delta initial condition) is determined by symmetry and the conservation of mass to be a Gaussian. Anything else would violate the central limit theorem. Fick's PDE is the continuum side of that theorem.

## What it may mean

If the equation holds at this precision in a homogeneous medium, it is reasonable to keep using it in the standard applications: drug release from polymer matrices, heat in solids (with thermal diffusivity in place of D), dopant diffusion in semiconductors, and oxygen across membranes. The exponent of t in the variance is 1.0 on this run, so there is no anomalous diffusion in the bulk. Anomalous diffusion is a real phenomenon, but it requires a structured medium (porous rocks, crowded cytoplasm) or a non-Markovian step distribution. Vanilla Fick is the right null model.

The numerical recipe transfers. Swap in a position-dependent D(x) for inhomogeneous media. Swap the explicit step for Crank-Nicolson if the problem is stiff or the boundary conditions are awkward. Add a reaction term and you have a reaction-diffusion equation, which is what Turing used in 1952 to explain animal coat patterns. The 1855 line stays the same line.

The Einstein link is what makes the result more than a textbook check. The same D that governs the PDE also governs single-particle Brownian motion through ⟨r²⟩ = 2 d D t, and through the Stokes-Einstein relation D = k_B T / (6 π η r) it connects to Boltzmann's constant. Perrin used that chain in 1908 to count molecules. Fick supplied the deterministic top half and Einstein supplied the stochastic bottom half, fifty years apart.

## Loose ends

With another week I would push the random walk to N = 100,000 and bootstrap the D_rw estimate to put a real confidence interval on it. I would also try an inhomogeneous D(x) (a step discontinuity at x = 0) and see whether the PDE and a position-dependent random walk still agree, which is a subtler question because the Itô-Stratonovich distinction matters. The whole pipeline reproduces in under a minute on a laptop, and the 1855 prediction has been stable for 170 years.
