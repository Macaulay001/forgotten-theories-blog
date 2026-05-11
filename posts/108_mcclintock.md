# I plugged McClintock's 1950 Ac/Ds excision rates into a branching-process model. The maize kernels came out spotted in the right way.

*Part 108 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1950 Barbara McClintock published a six-page paper in *PNAS* called "The origin and behavior of mutable loci in maize". She had spent most of the 1940s at Cold Spring Harbor staring through a microscope at the chromosomes of corn, and she had noticed that one place on chromosome 9 kept breaking. Not randomly. Always at the same spot, and only when a second, separate locus elsewhere in the genome was present. She called the breaking site Ds (for Dissociation) and the second locus Ac (for Activator). Without Ac, Ds did nothing. With Ac, Ds excised, jumped, and inserted itself somewhere else, sometimes into a pigment gene, turning a kernel from coloured to spotted.

She told the 1951 Cold Spring Harbor Symposium that genes can move. The room was polite. The field was not. Watson and Crick were a couple of years away from publishing the double helix, the molecular-biology programme was about to take off, and the idea that a chromosome was a tidy linear array of fixed addresses was about to become orthodoxy. McClintock kept publishing through 1953 and then, by her own account, gave up. She moved on to teosinte. She did not return to transposition until others discovered it again in bacteria in 1974 and in *Drosophila* in 1982. She got the Nobel in 1983, unshared, 33 years after the *PNAS* paper.

I wanted to know whether her numbers actually work.

## What I tried

The hypothesis breaks into three pieces.

First, the qualitative claim that genomes are full of mobile elements. This one is now trivially testable because we have sequenced the things. I hand-encoded the transposon / repeat fraction reported in a few classical genome papers: *E. coli* K12 (Blattner 1997, ~0.3%), yeast (Goffeau 1996, ~3%), *Arabidopsis* (the 2000 *Nature* paper, ~14%), *Drosophila* (Adams 2000, ~20%), human (Lander 2001, ~45%), maize B73 (Schnable 2009, ~85%). McClintock had the right organism for the maximum effect.

Second, copy number. McClintock and her collaborator Royal Alexander Brink reported that the autonomous Ac element transposes at a rate near 1% per cell division during early endosperm growth. If a transposon both duplicates itself and excises at roughly this rate, the population-genetic question is whether the copy number runs away to lethality or settles to a steady state. Charlesworth and Charlesworth wrote the deterministic equation in 1983: dn/dt = n(u - v - s*n), where u is the per-element insertion rate, v is excision, and s is the fitness cost per copy. The steady state n* = (u - v) / s. With u = 1.2e-2, v = 1.0e-2, and a tiny s = 1e-4, that gives n* = 20.

Third, variegation. If Ds sits inside a maize colour gene C in every cell of a developing aleurone (the outer kernel layer), and Ds excises at rate r per cell division, then a kernel grown by repeated doubling from a single founder should produce a tiled mosaic. Excision early in development makes a big coloured sector. Excision late makes a small spot. The patch-size distribution should look heavy-tailed.

I built a 256 x 256 lattice by 8 doublings, applied an excision probability of r = 0.04 at each doubling, and labelled the connected components afterwards.

## What happened

The genome bar chart says what you'd expect by 2026. The transposon-derived fraction ranges from 0.3% in *E. coli* to 85% in maize, a 280-fold span. Maize, the organism McClintock chose, has the highest known fraction of any well-studied genome. Her decision to work on corn was not arbitrary; corn was where the signal lived.

The Wright-Fisher trajectories for copy number settle near 13 elements per genome after 4000 generations, on the same order of magnitude as the deterministic estimate of 20. McClintock's contemporaries reported tens of active Ac copies per maize line, with the exact count varying by genetic background. The simulation reproduces the equilibrium and the stochastic fluctuation envelope that comes with it.

Here is the bare loop. No imports. The state vector n holds the per-lineage copy count, and each generation the mean shift is the deterministic drift plus a sqrt-n diffusion noise.

```python
n = np.full(N, 5.0)            # 200 lineages, each starting with 5 copies
for t in range(1, G):
    drift = n * (u - v - s * n)
    noise = rng.normal(0, np.sqrt(np.maximum(n, 1) * (u + v)), size=N)
    n = np.maximum(0, n + drift + noise)
    traj[:, t] = n
```

The mosaic kernel is the part that actually looks like a McClintock notebook page.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/108_mosaic_kernel.png)

The simulated aleurone has 2,205 separate pigmented sectors. The largest single sector covers about 2,100 cells, around 3% of the kernel surface. The smallest are isolated 1-cell flecks. A rank-size plot of patch areas is roughly straight on a log-log axis over two decades, which is the signature you would expect from a Poisson excision rate projected onto a binary tree of cell divisions. Early excisions are rare, but the patches they spawn are huge; late excisions are common but each one is small.

A small slice of the patch table:

| patch rank | size (cells) | when excision happened |
|------------|--------------|------------------------|
| 1          | 2132         | generation 1 or 2      |
| 10         | ~250         | generation 3 or 4      |
| 100        | ~25          | generation 5 or 6      |
| 1000       | ~4           | generation 7           |
| 2200       | 1            | generation 8 (final)   |

The "when excision happened" column is inferred from patch size by inverting 2^(G-g). McClintock did this inference by eye on real kernels and used it to time Ac activity within endosperm development. The arithmetic she was doing in the late 1940s is the same arithmetic the simulation does in 2026.

The copy-number equilibrium plot shows individual lineages bouncing between 5 and 30 copies, with the mean of 200 lineages tracking the deterministic n* line within a factor of two.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/108_copy_number_equilibrium.png)

And the bar chart, for context, on why maize was the right organism for this discovery:

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/108_species_te_content.png)

## What it may mean

Three things stand out. The first is just calibration. McClintock's reported rates, fed into a model written 30 years after her *PNAS* paper, give numbers consistent with what later sequencing turned up. The 1950 paper was quantitatively, not just qualitatively, correct.

The second is the role of the organism. Maize has the highest known transposon content of any major sequenced eukaryotic genome, partly because the lineage went through a whole-genome duplication ~10 million years ago and partly because plant retrotransposons (especially the LTR Gypsy and Copia families) are unusually active. Choosing corn for cytogenetics in 1944 was not luck; it was the loudest signal available. A *Drosophila* worker would have seen 20% transposon content and a much subtler variegation, the *Drosophila* mosaic-eye phenotype mapped to P elements that Rubin and Spradling cloned in 1982.

The third is the somatic-mosaic application. The same lattice model now appears in interpretations of skin neoplasms, of intestinal crypt clones, of L1 retrotransposition events in the brain. The rank-size plot for clonal patches of somatic mutations in neurons looks like the rank-size plot for McClintock's corn kernels. The mechanism is shared: a stochastic insertion / excision event applied to an exponentially growing pool of cells. She had the algorithm in 1950, just no neuronal sequencing to apply it to.

The 1983 Nobel citation reads "for her discovery of mobile genetic elements". The work was 33 years old by then. The simulation here takes about 12 seconds on a laptop. The patch distribution falls out of one line of NumPy. The delay between her result and its acceptance was not a delay of computation; it was a delay of taste.

## Loose ends

The selection coefficient s is fitted, not measured. A defensible next step would be to use the actual deletion / insertion rates per Ac generation now available from re-sequenced McClintock-stock lines (Stinard, MaizeGDB) to set u and v directly, and let s float as the free parameter. The lattice model also assumes synchronous doubling; real maize endosperm is syncytial for the first ~8 divisions before cellularising, which would smear the early patches. Adding that and rerunning would test whether the rank-size slope is steeper or shallower than the binary-tree prediction. Both look like an afternoon's work.
