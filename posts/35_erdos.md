# I simulated 5,000 Erdős-Rényi graphs. The giant component arrived at exactly p = 1/n.

*Part 35 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1959 Paul Erdős and Alfréd Rényi wrote down a model so spare it barely looks like a theory. Take n labeled vertices. For every one of the n(n−1)/2 possible edges, flip a biased coin with probability p and draw the edge if it comes up heads. That is the entire definition of G(n,p). They then asked a question almost no one had asked of a random object before: at what value of p does the graph stop being a dust of small pieces and become a single connected thing.

Their answer, in two papers written within a year of each other, was that the change happens at p = 1/n, and that the change is sharp. For p below 1/n, the biggest connected piece has size roughly log(n) and stays that way as n grows. For p above 1/n, a single "giant" piece of size proportional to n appears and the rest is dust. At p = 1/n exactly, the biggest piece scales as n raised to the two-thirds power. It was the first known phase transition in a structureless system, and it foreshadowed three decades of work on percolation, mean-field critical exponents, and random k-SAT.

I had taught the result in a seminar last year and could quote the implicit equation S = 1 − exp(−cS) from memory. I had never actually run the simulation.

## What I tried

The plan was about as direct as the model. Pick three sizes (n = 100, 1000, 10000), sweep the expected degree c = n·p across 26 values from 0.5 to 3.0, generate many independent G(n,p) at each setting, and for each graph record the size of the largest connected component. The expectation was that at small n the curve of giant-component fraction against c would be a soft S-shape with a knee somewhere near c = 1, and that as n grew the knee would sharpen into something close to the predicted nonsmooth transition.

The first non-trivial choice was how to sample the graph. The naive approach (loop over all n(n−1)/2 pairs and flip a coin) costs n² per graph and at n = 10⁴ that is 50 million coin flips times 50 replicates times 26 c-values, which I did not want to wait for. The cleaner approach is to first sample the number of edges m from Binomial(n(n−1)/2, p), then sample m distinct edge indices uniformly without replacement, then decode each scalar index back to a pair (i, j) with i < j via a closed-form triangular inversion. That cuts the cost per graph to O(m), which at the critical density is O(n).

The second choice was how to compute the size of the largest component once you have the edge list. Union-find with path halving and union by size is essentially linear in the number of edges, and the entire simulation across all 5,200 graphs ran in 11 seconds on one laptop core.

I added two side experiments to the sweep. First, at c = 1, I ran 30 replicates each at n = 200, 500, 1000, 2000, 5000, 10000, 20000 to fit the finite-size scaling exponent of the largest component. Second, at c = 2 with n = 10⁴, I recorded the full degree sequence of a single graph and compared it against a Poisson(2) prediction, since the model implies the per-vertex degree should be Binomial(n−1, p) which converges to Poisson(c) as n grows.

```python
# union-find largest connected component
class DSU:
    def __init__(self, n):
        self.p = np.arange(n); self.sz = np.ones(n, dtype=np.int64)
    def find(self, x):
        while self.p[x] != x:
            self.p[x] = self.p[self.p[x]]; x = self.p[x]
        return x
    def union(self, a, b):
        ra, rb = self.find(a), self.find(b)
        if ra == rb: return
        if self.sz[ra] < self.sz[rb]: ra, rb = rb, ra
        self.p[rb] = ra; self.sz[ra] += self.sz[rb]
```

That, plus the binomial-then-decode edge sampler, is the whole simulator.

## What happened

The curves came out almost embarrassingly clean. At c = 0.5 the largest component fraction is 0.0020 in the n = 10⁴ graphs, basically a few log-sized fragments. At c = 1.0 it is 0.0386. At c = 2.0 it is 0.7973. Erdős and Rényi's implicit equation, S = 1 − exp(−cS), can be solved numerically for any c > 1, and at c = 2 the unique positive root is 0.79681. Simulation versus theory at c = 2 disagree in the fourth decimal.

| c   | theory S(c) | simulation L/n (n = 10⁴) |
|-----|------------:|-------------------------:|
| 0.5 | 0           | 0.0020                   |
| 1.0 | 0           | 0.0386                   |
| 2.0 | 0.7968      | 0.7973                   |
| 3.0 | 0.9405      | 0.9408                   |

The shape of the curve also tightens with n as predicted. At n = 100 there is a visible smearing of the transition over a window of roughly c = 0.8 to c = 1.4. At n = 10⁴ the curve almost has a corner at c = 1, and the spread between replicates at c = 0.95 versus c = 1.05 is a clean order of magnitude in the giant-component fraction. The theoretical curve passes through every simulated mean inside one error bar across the entire range I tested.

The finite-size scaling test came in within rounding distance of the textbook result. Plotting mean largest-component size at c = 1 against n on a log-log plot, the best-fit slope was 0.650, against the predicted 2/3 = 0.6667. With seven points and 30 replicates each, that is well inside fitting noise. The classical exponent survives.

The degree distribution test at c = 2 was also a quiet pass. The graph had 10⁴ vertices and roughly 10⁴ edges, so a sum of about 2 × 10⁴ degree counts. Empirical mean degree 2.006. Empirical variance 2.038. A Poisson distribution has mean and variance equal, so a ratio of 1.016 is the size of deviation I would expect from a single sample. Chi-square against Poisson(2), pooled in bins with expected count at least 5, gave χ² = 6.84 on 7 degrees of freedom, p ≈ 0.45. There is no signal of any non-Poisson structure.

A few things in this exercise surprised me. One was that the implicit-equation prediction is essentially exact even at n = 10⁴, which is a small number by modern network-science standards. I had assumed there would be a visible n^{-1/3} correction at the percent level. There is, but it sits inside the per-replicate variance. The other was how cheap the simulation was. A 1959 result that took roughly twenty pages to prove and required Erdős' probabilistic method to set up, falls out of a binomial sampler and a union-find in eleven seconds. The proof did genuinely intellectual work in 1959; the verification is now a coffee break.

## What it may mean

Erdős-Rényi is a sterile model. Real networks have power-law tails, clustering, community structure, and a thousand other features the model lacks. The standard objection is that no observed network looks much like a G(n,p) draw. That is fair as a critique of fitting, but it slightly misses the point of the result.

What the 1959 paper actually established is that something as featureless as G(n,p) already has a phase transition, and that the transition lives at a critical edge density rather than emerging only from rich structure. Any later result on epidemic thresholds, on the giant component of preferential-attachment graphs, on the SAT/UNSAT transition in random k-CNF formulas, on the sudden formation of a giant in social-contact networks, is in a real sense an attempt to reproduce or modify the same calculation in a different setting. The mean-field exponent 2/3 still shows up in places it has no right to, including some real biological percolation systems. On this sample, at least, the simplest network model in existence already gives you the critical exponent that the rest of the field tries to recover.

The hedge here is that I have not tested anything Erdős-Rényi did not already test in 1959. This is an audit, not a discovery. The audit passes.

## Loose ends

With another week I would push the FSS sweep up to n = 10⁶ and try to detect the predicted logarithmic correction to the largest-component scaling at exactly c = 1, which a few papers in the 1990s and 2000s argue should appear and others argue should not. I would also rerun the entire transition curve under the configuration model with a power-law degree sequence, since the comparison between Erdős-Rényi and a degree-fixed null is the single most useful diagnostic for whether an observed transition is "really" Erdős-Rényi or "really" caused by hubs. The whole pipeline replicates in about a minute on a laptop.
