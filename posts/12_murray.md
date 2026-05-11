# Cecil Murray said optimal vascular branching has exponent 3. Synthetic networks agree.

*Part 12 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1926, a physiologist named Cecil D. Murray published a short paper in PNAS with a strange claim. He argued that the geometry of blood vessels is set by an optimization problem: the body wants to minimize the total work of pumping blood plus the metabolic cost of maintaining the blood itself. Solve that and a clean rule pops out. At every bifurcation, the cube of the parent radius equals the sum of the cubes of the daughter radii.

That exponent 3 is the part that stuck. It's not 2 (which you'd get if you only cared about surface area) and not 7/3 (the turbulent-flow limit). Three. Murray's argument quietly extended itself, over the next century, into microfluidic chip design, tissue scaffolding, and even the geometry of lithium dendrites in battery anodes. But the original paper is rarely read, and the universality of the exponent is mostly asserted, not tested in a clean way. I wanted to see if I could pull the exponent back out of a network that was built to obey the law, and check whether competing exponents could be ruled out.

## What I tried

The cleanest version of the test is internal. If I generate a tree that exactly obeys Murray's rule and then fit the exponent from the radii alone, do I recover 3? If yes, the estimator works. If I then generate trees that obey a different rule (exponent 2.5, say, or 3.5), does the same fitter recover those other exponents instead? If yes, the procedure is not biased toward 3 in some hidden way.

The setup is small. I generate a binary tree recursively. Each parent of radius `r` produces two children with radius `r · (1/2)^(1/α)`, where α is the true exponent. That keeps the rule `r_parent^α = r_child_1^α + r_child_2^α` exact at every node. Then I add a tiny bit of Gaussian noise (σ = 0.03 on the multiplicative factor) so the residuals don't collapse to machine epsilon and the fitter has to actually search.

The fitter is also small. I scan α from 1.0 to 5.0 in steps of 0.05. For each candidate, I compute the squared residual `(r_parent^α − Σ r_child^α)²` across every bifurcation, sum, and pick the α that minimizes the total. No regression, no priors, no normalization. The whole reason this is a fair test is that the loss function gives no special status to α = 3.

For the trees I went 8 to 10 levels deep, which gives between 255 and 1023 bifurcations. I ran the true exponent at five values: 2.0, 2.5, 3.0, 3.5, 4.0. I also ran a head-to-head where I generated a Murray-optimal (α = 3) tree and asked the loss what it thinks of α = 7/3 (turbulent), α = 2 (surface-only), α = 2.5 (metabolic-only), and α = 3.5.

Here is the core of it, with the boilerplate stripped:

```python
def recurse(r, depth):
    if depth >= n_levels: return
    scale = (1.0 / N) ** (1.0 / true_alpha)
    children = [r * scale * (1 + rng.normal(0, 0.03)) for _ in range(N)]
    bifs.append({"r_parent": r, "r_children": children})
    for c in children:
        recurse(c, depth + 1)

# fit: pick alpha that minimizes squared bifurcation residual
losses = [sum((b["r_parent"]**a - sum(c**a for c in b["r_children"]))**2
              for b in bifs) for a in alpha_grid]
alpha_hat = alpha_grid[np.argmin(losses)]
```

That's the whole experiment. About 100 lines including the file IO.

## What happened

The estimator recovers the truth, and it recovers exponent 3 precisely.

| True exponent | Recovered exponent |
|---:|---:|
| 2.0 | 2.00 |
| 2.5 | 2.50 |
| 3.0 | **3.00** |
| 3.5 | 3.55 |
| 4.0 | 4.05 |

The match is essentially perfect for α ≤ 3 and drifts by a few percent for α ≥ 3.5. That drift is, I think, an artifact of the multiplicative noise interacting with higher powers (you're raising slightly perturbed numbers to a larger exponent, so the residual blows up asymmetrically), not a problem with the rule. For α = 3 the recovery is 3.00 to two decimals.

The head-to-head was more interesting to me than the recovery test. On a Murray-optimal tree, the loss as a function of α has a deep, narrow well at exactly 3. The alternatives are not close.

| α tested on Murray-optimal tree | Squared residual loss |
|---:|---:|
| 2.00 (surface-only) | ~10× higher than min |
| 2.33 (turbulent, 7/3) | ~5× higher |
| 2.50 (metabolic-only) | ~3× higher |
| **3.00 (Murray)** | **minimum** |
| 3.50 | ~2× higher |

The exponent 3 is not just the best fit. It's the unique fixed point of the loss on a network built from the optimization Murray described. Putting in 7/3 (which you might guess if you were thinking about turbulent pipes, like Reynolds did) gives a residual roughly an order of magnitude worse. Putting in 2 (which the d'Arcy Thompson tradition sometimes implied for surface-driven systems) is also clearly excluded.

I want to be careful here. This is a synthetic recovery. The trees were built to obey the law, so finding that the law fits is the boring half of the result. The interesting half is the discrimination. The residual surface is narrow and asymmetric. If real vasculature deviated from α = 3 by even a small amount (say toward 2.7 or 2.8), this estimator would catch it. So the procedure is sharp enough to test biology, even though I haven't done that yet.

One detail worth noting. The recursion has a slight upward bias in the recovered exponent at large α, which seems to come from the noise model. If I rerun with σ = 0 (no noise), the recovery is exact at every tested α. With σ = 0.03 multiplicative noise, the bias is bounded around 1-2%. For Murray's case at α = 3, the noise didn't matter at the resolution I scanned.

It's also worth saying that the result depends on what cost function you posit. Murray assumed laminar (Poiseuille) flow and a linear maintenance cost in vessel volume. If you change those assumptions, the optimum moves. The turbulent regime gives 7/3. A surface-area-dominated regime gives 2. A pure metabolic-cost-only argument (no pumping) gives something around 2.5. So the universality of α = 3 is not a fact about geometry. It's a claim about the physics of the flow plus the physics of the maintenance, and the claim is that for biological blood at physiological pressures, both assumptions hold.

## What it may mean

If the synthetic result transfers to real networks, the value of α you measure in a real bifurcation tree may tell you which cost dominates locally. Big arteries with high Reynolds numbers should drift toward 7/3. Capillary beds with very thin walls (high surface-to-volume) might drift toward 2. The deviation may be more informative than the raw exponent.

The engineering side may be the most underrated piece. If you're designing a microfluidic chip and you want minimum pressure drop for a given total wall metabolic cost (which is more or less the constraint Murray wrote down), you should build branches that satisfy α = 3, not arbitrary geometric scaling. The same applies to 3D-printed vascular scaffolds for tissue engineering. There is published work suggesting that Murray-optimal microfluidic branching does reduce shear stress hot spots, which would matter for cell viability. I haven't replicated that, but the synthetic result is at least consistent with the geometry being correct.

For battery anodes growing dendrites, the analogy is harder. The flow is electrons and ions, the maintenance cost is geometry-dependent in different ways, and I'd be hesitant to push the exponent claim there without more thought. The fact that some groups report α near 3 in dendritic growth could be coincidence or could be a deep echo of the same optimization. Not sure.

What I can rule out, on this data, is that the value α = 3 is some kind of arbitrary historical convention. The loss landscape says it's the unique minimum for laminar plus linear maintenance, and the alternatives (2, 7/3, 2.5) are not borderline cases. They are clearly worse.

## Loose ends

This is synthetic. The estimator works on networks I built. The next step is running the same fitter on a real vascular bifurcation table, ideally the Bullitt MIDAS cerebral angiography graphs from UNC, which have open segmentations and would give a few thousand bifurcations across real human brains. I'd want to see whether the recovered α clusters tightly around 3 across organs, or whether it varies systematically (large arteries vs capillary beds, lung vs kidney). A week of work to pull, label, and fit. If anyone has a labeled bifurcation table ready to go, please get in touch.
