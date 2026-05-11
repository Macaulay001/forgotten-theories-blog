# I rebuilt Kauffman's 1989 NK landscapes. The sweet spot for evolvability sits at K=2.

*Part 26 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1989 Stuart Kauffman published a model of fitness landscapes called
NK. N genes, each one influenced by K other genes, each combination
of bits assigned a random fitness contribution. The model has one knob,
K, and turning it changes the texture of the search space from a smooth
single-peaked hill (K=0) to a maximally rugged hash (K=N-1) where every
mutation jumps to an unrelated fitness value.

Kauffman's claim was twofold. Intermediate K, somewhere around 2 to 4,
gives populations the best of both worlds: enough structure to climb but
enough roughness for multiple accessible peaks. And at the far end,
where K reaches N-1, there is a "complexity catastrophe", the local
optima you can reach all collapse toward the mean of the random
fitness distribution. Climb as hard as you like, you end up near 0.5.

The model became famous, then sort of background furniture. Modern
biology rarely cares about K=N-1, and the intermediate-K claim got
folded into evolvability theory without people rerunning the original
calculation. I wanted to see how cleanly the curve still falls out of a
clean implementation in 2026.

## What I tried

The NK setup is small enough to write in an afternoon. For each locus i,
pick K other loci it depends on. Build a lookup table of size 2^(K+1)
with random uniform fitness contributions. Total fitness is the average
contribution across all N loci. An adaptive walk is greedy steepest
ascent: flip one bit, keep the best improving neighbor, stop when no
neighbor improves.

I used N=20, K in {0, 1, 2, 4, 8, 19}, and ran 200 walks per K spread
across 5 independently sampled landscapes. The walks start from random
genomes and terminate at the first local optimum they find. For each
walk I logged the starting fitness, the final fitness, and the number
of accepted improving steps.

The whole thing runs in under five minutes on one CPU. No GPU, no
parallelism. I deliberately kept the implementation close to Kauffman's
original prose so the numbers would be directly comparable to anyone
who wants to redo it.

Here is the core of the climb:

```python
def adaptive_walk(genome, neighbors, table):
    g = genome.copy()
    f = fitness(g, neighbors, table)
    while True:
        best_f, best_i = f, -1
        for i in range(len(g)):
            g[i] ^= 1
            f_new = fitness(g, neighbors, table)
            if f_new > best_f:
                best_f, best_i = f_new, i
            g[i] ^= 1
        if best_i == -1:
            return g, f
        g[best_i] ^= 1
        f = best_f
```

That is the whole engine. The fitness function reads the K+1 relevant
bits at each locus, packs them into an integer index, looks up the
contribution, averages across N loci. Nothing exotic.

## What happened

The curve I got back is the textbook Kauffman shape, but it felt good
to see it land cleanly.

| K | mean final fitness | sd | walk length | gain per step |
|---|---|---|---|---|
| 0 | 0.689 | 0.048 | 10.0 | 0.019 |
| 1 | 0.696 | 0.042 | 8.8 | 0.023 |
| **2** | **0.740** | 0.031 | 7.6 | 0.031 |
| 4 | 0.718 | 0.047 | 5.5 | 0.042 |
| 8 | 0.694 | 0.035 | 3.4 | 0.063 |
| 19 | 0.642 | 0.027 | 1.6 | 0.105 |

K=2 wins the final-fitness contest. K=0 is the classic smooth landscape:
one global peak, every walk eventually finds it, but the peak itself
sits at 0.689 because averaging 20 independent uniform draws caps you
in that neighborhood. K=2 walks reach 0.740 on average, a roughly 7%
improvement over K=0 and a 15% improvement over K=19. The K=2 vs K=0
gap is around 9 standard errors apart on this sample, so it is not noise.

The walk-length column tells the structural side of the story. Smooth
landscapes (K=0) let you climb for about 10 steps before you exhaust
the improving moves. Maximally rugged ones (K=19) trap after one or two
steps because almost every neighbor has a randomly redrawn fitness and
is more likely worse than better. The per-step gain compensates a bit at
high K (each rare improving move buys more) but the total trip is too
short to overcome the cliff.

The complexity catastrophe sits at K=19, where final fitness drops to
0.642. Order-statistic theory predicts roughly 0.66 for the maximum over
N+1 random points in a fully decorrelated landscape, and our number
sits a hair below that, which makes sense because greedy walks do not
always find the best neighbor of their start when later moves redraw
the table. The catastrophe is real but mild: 0.642 is still well above
the random expectation of 0.500. You cannot fail to climb at all, you
just cannot climb far.

What I did not expect was how strongly the per-step gain rises with K.
At K=19 each accepted move adds 0.105 in fitness, against 0.019 at K=0.
If a system could be confident it had an improving move available, high
K would be the best place to be, but the count of such moves dries up so
fast that it is a Pyrrhic gradient.

## What it may mean

The intermediate-K optimum is the part of Kauffman's claim that has aged
best. Empirical fitness landscapes for proteins and small RNAs come out
with K of roughly 1 to 5 when fit to NK-like models, which puts real
biology right where this simulation says evolvability is strongest. That
is consistent with the long-running observation that protein active
sites have moderate epistasis but not extreme epistasis. A maximally
modular protein (K=0) would be evolvable but boring, a maximally coupled
one (K=N-1) would be a frozen hash.

The catastrophe end of the claim has aged less well, not because it is
wrong but because nobody actually lives there. Real molecular networks
seem to avoid K=N-1 regimes the way water avoids 100°C unless you push
it. So the catastrophe may be a thing organisms have learned to skirt
rather than a thing they crash into.

For engineered systems, the lesson may be sharper. Combinatorial drug
discovery, neural-architecture search, organizational network design,
anywhere a coupling parameter exists, K=2 to K=4 appears to be the
default place to start. Lower means you find the obvious optimum and
stop. Higher means you wander into trap-rich territory and get pinned
near the mean.

This is not a proof that K=2 is universally best. The NK formulation
assumes uniform fitness contributions and one specific neighbor-picking
rule. Real systems probably have heavy-tailed contributions and
correlated dependencies. Still, the qualitative shape, peak in the
middle, falls out of so many variations of the setup that the core
phenomenon seems robust.

## Loose ends

Five minutes of compute is not enough for a clean test of the K=2 vs
K=4 race. With N=20 they overlap within one standard deviation. A larger
N (say 60 or 100) would sharpen the peak and likely shift it slightly,
since the optimum K depends weakly on N. I also ran only steepest-ascent
walks; stochastic walks and long-jump search flatten the K dependence
because they can escape local traps. If anyone has a curated empirical
landscape with N>30 and bit-flip fitness data, I would love to fit
the NK K parameter to it directly. The simulation above replicates in
about ten minutes including the plotting script.
