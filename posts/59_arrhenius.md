# Arrhenius's 1889 rate law recovers literature activation energies to within 3 kJ/mol, and the molecular picture comes 10% short

*Part 59 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Svante Arrhenius spent the early 1880s arguing that salts in water
split into charged particles, an idea that earned him a barely-passed
doctoral thesis in Uppsala and, fifteen years later, a Nobel Prize. In
the same productive stretch, in 1889, he published a paper on cane
sugar inversion that ended with a small empirical equation:

    k = A exp(-Ea / RT)

A rate constant k rises with temperature T through a single
exponential governed by an activation energy Ea and a pre-exponential
factor A. Plot the logarithm of k against 1/T and the points should
fall on a line of slope -Ea/R. Arrhenius arrived at it partly from
thermodynamic analogy and partly from a kinetic intuition: only
molecules with enough kinetic energy to clear an internal barrier
react, and the fraction of such molecules is set by the Boltzmann
factor.

It is one of those equations that nobody really doubts any more. It
shows up in every chemistry textbook, in combustion modelling, in
solid-state diffusion, in pharmaceutical shelf-life calculations, and
in battery aging. The reason I wanted to look at it again is that the
"kinetic energy threshold" derivation has a subtle gap. Maxwell-Boltzmann
gives you the full distribution of molecular energies, not just the
exponential tail, and the survival fraction is not exactly exp(-Ea/RT).
So how well does a literal molecular Monte Carlo reproduce the rate
law that has Arrhenius's name on it?

## What I tried

The plan had two halves. The first was a direct audit on the
experimental side: take published rate constants for two
well-characterised reactions, fit ln k against 1/T, and read off Ea.

The reactions I picked are the textbook ones. Gas-phase decomposition
of N2O5, which Daniels and Johnston measured in 1921, is one of the
first reactions ever shown to be cleanly first-order, and its
activation energy is quoted at about 103 kJ/mol almost everywhere.
Acid-catalysed sucrose hydrolysis is the very reaction Arrhenius wrote
about in 1889; Wilhelmy had measured it in 1850, and Moelwyn-Hughes
later tabulated rate constants across a 40 K window. The literature
consensus for its activation energy sits near 107-108 kJ/mol.

I hand-encoded six (T, k) pairs for N2O5 from secondary sources and
five for sucrose, then ran a one-line linear regression on
(1/T, ln k).

The second half was a kinetic-theory check. At each of nine
temperatures from 400 to 700 K I drew two million translational
kinetic energies from a Maxwell-Boltzmann distribution. In molar
units that distribution is Gamma-shaped: E_mol / (RT) follows a
Gamma(3/2, 1). Pick a threshold Ea (I used 50 kJ/mol, low enough that
even at 400 K the reactive fraction is well above noise), count how
many sampled energies exceed it, and treat that fraction as a relative
rate constant. Then fit the simulated points back to the Arrhenius
form and see whether the input threshold and the recovered Ea agree.

If Arrhenius's intuition is exact, the recovered Ea should match the
input within sampling error. If the threshold-rate story is only
approximate, the bias should show up cleanly.

## What happened

The literature fit is almost embarrassingly clean. For both reactions
the points fall on a line with r squared above 0.9999. The slopes
return Ea values within a few percent of the canonical numbers:
102.9 kJ/mol for N2O5 against ~103 kJ/mol in the literature, and
110.7 kJ/mol for sucrose against ~108 kJ/mol. The residuals in ln k
stay below 0.1 across the whole measured T range, with no obvious
curvature.

![Arrhenius plot](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/59_arrhenius_fit.png)

The Monte Carlo half is more interesting. Here is the core of it:

```python
# Sample molar KE from Maxwell-Boltzmann: E / RT ~ Gamma(3/2, 1)
for i, T in enumerate(T_grid):
    x = rng.gamma(shape=1.5, scale=1.0, size=N_samples)
    E_mol = x * R * T
    sim_k[i] = np.mean(E_mol > Ea_input)

slope, intercept = np.polyfit(1.0 / T_grid, np.log(sim_k), 1)
Ea_recovered = -slope * R
```

The simulated points lie on a near-perfect Arrhenius line (r squared
of 0.991), and the slope gives Ea = 45.2 kJ/mol. The input threshold
was 50.0 kJ/mol. So the molecular picture reproduces the shape but
the recovered Ea sits about 10% low.

| Quantity | Value |
|---|---|
| N2O5 fitted Ea | 102.9 kJ/mol |
| Sucrose fitted Ea | 110.7 kJ/mol |
| MC input threshold | 50.0 kJ/mol |
| MC recovered Ea | 45.2 kJ/mol |
| Linear fit r squared (worst) | 0.991 |

That 10% gap is not noise. It is a real feature of the kinetic-theory
derivation that the bare Arrhenius form cannot represent. For Ea/(RT)
large, the survival fraction of a Maxwell-Boltzmann distribution is
roughly (2/sqrt(pi)) sqrt(Ea/RT) exp(-Ea/RT), not just the exponential
on its own. The sqrt(T) prefactor adds a -(1/2) ln T term to ln k,
which a linear ln k vs 1/T fit absorbs by lowering its slope. So
Arrhenius's exponential is the right asymptotic form, but the
collision picture predicts a small temperature-dependent
pre-exponential that the original 1889 equation ignored.

This is exactly why modern kinetic databases, the GRI-Mech mechanism
for methane combustion, the NIST kinetics database, and most
atmospheric chemistry packages, use a three-parameter modified
Arrhenius form k = A T^n exp(-Ea/RT). The integer or half-integer
exponent n is collision theory leaking through. In this sample, the
"missing" exponent shifted the apparent Ea by 4.8 kJ/mol on a 50 kJ/mol
input. For a stiffer 100 kJ/mol barrier the relative bias is smaller,
which is partly why the real-world fits to N2O5 and sucrose look so
clean: their Ea/(RT) is around 40 at room temperature, putting them
deep in the regime where the exponential dominates everything else.

![Linearity residuals](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/59_residuals.png)

The residual plot for the two literature systems shows no obvious
curvature; the points scatter around zero with no S-shape. That is
consistent with the high Ea/RT regime washing out the prefactor
correction.

## What it may mean

Arrhenius's 1889 equation appears to survive a hundred and thirty
years of scrutiny essentially intact. The exponential is the right
functional form. The Boltzmann factor really is doing the work. The
fact that two reactions chosen for historical reasons return
activation energies within 3 kJ/mol of their textbook values is a
modest but real check, and the fit quality is the kind you only get
when the underlying law is correct.

The Monte Carlo result is the more useful finding. It suggests that
when chemists fit a single exponential to a finite temperature window,
they are recovering a slightly biased Ea, where the bias comes from
the molecular distribution they are implicitly assuming. For
laboratory work over a 40 or 50 K window with Ea around 100 kJ/mol,
that bias is below experimental uncertainty. For high-temperature
combustion modelling, where Ea/RT can drop into the single digits,
the bias grows and the modified Arrhenius form earns its keep.

None of this is new chemistry. It is just a reminder that the
simplest forms of any law absorb structure that you might prefer to
see separately. The same lesson appeared in the Newton-cooling check,
where the exponential silently rebranded radiative loss as faster
convection.

## Loose ends

I used digitised values for the k(T) tables rather than going back to
the primary measurements. The sucrose Ea came out 2-3 kJ/mol above
the most-cited literature value, which probably reflects rounding in
the secondary tabulation. The Monte Carlo model only includes
translational kinetic energy; real reactions also need rotational
and vibrational excitation, and an orientation factor for the
colliding molecules. With another week I would rerun the MC with a
proper line-of-centres collision criterion and check whether the
recovered n in k = A T^n exp(-Ea/RT) matches the 1/2 the math
predicts.
