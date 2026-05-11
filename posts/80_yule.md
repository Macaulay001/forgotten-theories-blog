# Yule's 1925 species-per-genus law reproduced in a 100,000-step simulation

*Part 80 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1925, the statistician G. Udny Yule sat down with a dataset
collected by the botanist J. C. Willis and wrote one of the first
mathematical models of evolution. Willis had been cataloguing species
per genus across plant families and noticing something strange. Most
genera contain very few species. A handful contain hundreds. The
distribution did not look like a bell curve. It looked like a
straight line on log-log paper.

Yule's question was not "is this curve real" but "what simple
process could produce it". His answer, in *A Mathematical Theory of
Evolution*, was a stochastic recipe with two knobs. At each step in
evolutionary time, with probability p a brand-new genus is founded
with a single species. Otherwise, a new species is added to an
existing genus, and the existing genus is chosen with probability
proportional to its current size. Big genera attract new species
faster than small ones, because they have more lineages from which
to throw off variants.

Yule worked out what the steady-state distribution of genus sizes
should look like under this rule. In the tail, where the math is
clean, the probability of seeing a genus with k species behaves like

    P(k) is proportional to k^(-rho),   with rho = 1 + 1 / (1 - p).

Three decades later Herbert Simon (1955) rederived the same thing
for word frequencies and other heavy-tailed counts, and the modern
name became the Yule-Simon distribution. The same arithmetic
resurfaced in 1999 as the Barabasi-Albert preferential-attachment
network. Yule had got there in 1925, in a different vocabulary, with
botany as the application.

The thing that struck me about the original paper is how compact the
mechanism is. Two lines of code is enough to write the dynamics
down. I wanted to actually run it and see whether the fitted tail
exponent comes out where Yule said it should.

## What I tried

The plan was a single-pass simulation in pure numpy. Start with one
genus containing one species. Each step, draw a uniform random
number. If it lands below p, append a new genus to the list with
size 1. If it lands above p, sample a species at random from the
"ticket pool" of all species seen so far and bump the size of the
genus that species belongs to. The ticket pool gives you
preferential attachment for free: a genus with twice as many species
has twice as many tickets in the urn.

I ran 100,000 steps for four values of p in {0.10, 0.25, 0.50, 0.75}
with a fixed seed, then fit the tail of each resulting size
distribution with the Hill maximum-likelihood power-law estimator on
k >= 5. The Hill MLE is

    rho_hat = 1 + n / sum( log(k_i / (kmin - 0.5)) )

which is the discrete-friendly variant from Clauset, Shalizi and
Newman (2009). Predicted exponents come straight from Yule's
1 + 1/(1-p) formula.

Independently I wanted to look at a small real species-per-genus
sample. I hand-encoded 145 counts spanning genera with a single
species up to a few outliers with 200-plus species. The numbers are
illustrative rather than a clean GBIF pull, but they have the right
heavy-tailed shape for a quick visual check.

The whole pipeline runs in under twenty seconds on a laptop. The
interesting question is not whether the qualitative shape comes out
right (it has to, by construction) but whether the fitted exponent
agrees numerically with Yule's prediction, and how the agreement
degrades as p climbs.

## What happened

The fits sit close to the predictions where the tail is long enough
to fit. Here is the simulation summary.

![Yule-Simon simulation](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/80_yule_simon.png)

| p | genera | max size | rho predicted | rho fitted |
|---|---:|---:|---:|---:|
| 0.10 |  9,985 | 45,658 | 2.111 | 2.078 |
| 0.25 | 25,188 |  8,788 | 2.333 | 2.196 |
| 0.50 | 50,098 |    926 | 3.000 | 2.727 |
| 0.75 | 75,198 |     31 | 5.000 | 3.949 |

At p = 0.10 the fit lands within 1.6 percent of Yule's prediction.
At p = 0.25 the gap opens to about 6 percent. By p = 0.75 the
empirical tail is so short, the biggest genus has only 31 species,
that the Hill estimator effectively has nothing to grip onto and
biases low. The plot of empirical PMF against the reference slope
makes this visible: each predicted line passes through the cloud of
points wherever the cloud extends far enough to define a tail.

Here is the core of the dynamics. No imports, no boilerplate.

```python
# Yule 1925: build the genus-size distribution
sizes   = [1]      # genus 0 starts with one species
tickets = [0]      # one ticket per species, value = genus index
for _ in range(n_steps):
    if rng.random() < p:
        sizes.append(1)
        tickets.append(len(sizes) - 1)
    else:
        g = tickets[rng.integers(0, len(tickets))]
        sizes[g] += 1
        tickets.append(g)
```

The "ticket pool" trick is the part Yule could not have implemented
the same way in 1925. It buys you exact preferential attachment in
O(1) per step instead of O(number of genera). The math is identical
to drawing a genus index with probability size_g / total, but the
sampling is much faster.

For the hand-encoded real dataset, the fit returned rho ~ 2.03 on
the k >= 5 tail. Inverting Yule's formula,

    p = 1 - 1 / (rho - 1) ~ 0.03,

which says the implied per-step rate of founding a wholly new genus
is only a few percent. That is in the right ballpark for what
Willis's own *Age and Area* tabulations suggested almost a century
ago. The sample is small enough that I would not push this number
hard, but it is consistent with the general picture that genus
founding is rare relative to within-genus speciation.

The systematic drift at large p is worth flagging. It is not that
Yule's formula is wrong. The mechanism is exact in the
N -> infinity limit. The drift comes from two finite-sample issues.
First, the Hill estimator with a fixed kmin underestimates rho when
the empirical tail is short and dominated by the bulk. Second, at
large p the rate of new genus founding is high enough that
individual genera do not get time to grow into a clean tail. Even
the largest genus at p = 0.75 has 31 species, so the "tail" the fit
sees is a few dozen integers wide.

## What it may mean

Yule's process is one of the cleanest demonstrations in statistics
of how a heavy-tailed distribution can fall out of a simple
proportional-growth rule plus a constant rate of new entrants. The
same arithmetic generates Pareto's wealth law, Zipf's word
frequencies, citation counts in academic literature, and the degree
distributions of scale-free networks. In each case the substrate is
different (people, words, papers, web pages, genera). The mechanism
is the same: a positive feedback in which size begets growth,
diluted by a steady trickle of new units starting at size one.

What I take from this little exercise is that the link between
mechanism and exponent is tight. If you observe a tail with rho
close to 2, the implied new-entrant rate is small (a few percent
per step). If you see rho close to 3, the rate is more like a half.
The slope tells you something about the underlying dynamics, not
just about the static shape. That inversion is what makes Yule's
1925 paper feel modern. He was reading off a parameter of an
evolutionary process from the slope of an empirical curve.

The caveats are still there. Real biology is not a Markov urn. New
genera do not appear by a flat per-step Bernoulli. Speciation rates
vary across clades, and some genera are pulled apart by taxonomists
rather than collapsed by extinction. None of that breaks the
qualitative story. It does mean that a fitted p from a single rho
should be read as a rough effective rate, not a literal probability.

## Loose ends

The real-data slice here is small and hand-encoded. The next step
would be a clean GBIF or Catalogue of Life pull of species counts
per genus, fit per plant family with the full Clauset-Shalizi-Newman
procedure (KS-optimal kmin, bootstrap uncertainty), and a check on
whether the implied per-family p tracks anything biologically
sensible like age of the family or geographic spread. With another
week I would also rerun the simulation at 10^7 steps for the high-p
cases so the Hill fits stop being truncation-limited. You can
replicate the 100,000-step version in about twenty seconds on a
laptop.
