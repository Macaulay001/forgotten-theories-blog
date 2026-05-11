# Why every cell appears to inherit a phosphorylated molecule, in 1500 lines of Python

*Part 1 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

## The strange old observation

In 1987 Frank Westheimer wrote a short paper called *Why Nature Chose Phosphates*. The argument was chemical: only phosphate has the right combination of negative charge, hydrolyzable bonds, and stability in water to do what life needs it to do (genetic backbone, energy currency, signalling). It's a famous paper, but it never quite answered the obvious follow-up. If phosphate is so special, can you see that specialness in the structure of metabolism itself, without invoking kinetics or selection pressure? Westheimer's case was about molecules in isolation. Metabolism is a network. Nobody (that I could find) had asked the network version of the question.

The thing that nagged at me was simpler. If you take a curated metabolic reconstruction of *E. coli* and put it on minimal medium, less than 4% of the network is reachable from food alone. The cell is not self-producing from the food set. It inherits something. What is the smallest thing it has to inherit?

## What I tried

The closure computation is almost trivial. You have a set of reactions, each with reactants and products, and a food set F. Compute the largest set of metabolites you can reach by repeatedly firing reactions whose reactants are already present. Polynomial time. The function in `src/seed.py` is about 20 lines and runs in milliseconds on an *E. coli*-sized model.

Given the closure, the question becomes: which single non-food metabolite, if you added it to F, would expand the closure the most? Call that the tier-1 keystone. Brute force is fine. For *E. coli* iML1515 there are about 1810 candidates, each closure runs in a few milliseconds, the whole sweep finishes in under a minute on one core.

I started with four organisms (BiGG iML1515, iMM904, iJN746, iIT341). The rank-1 keystone was phosphorylated in all four. That could be a coincidence at n=4, given that roughly 32% of candidate metabolites in these networks carry a phosphate atom. So I scaled it up.

The full BiGG collection has 108 models covering 85 unique organisms, from *E. coli* strains to *Plasmodium* to methanogenic archaea. That's still a small enough number to run by hand. Then I pulled EMBL-GEMs (CarveMe-generated, fully automated reconstructions, philosophically opposite to BiGG's hand curation) and sampled 400. Then AGORA, the manually curated gut microbiome collection, another 400. Different curation pipelines, different conventions for which metabolites count as currency, different reaction-balancing styles.

For each model I built three things. The tier-1 scan (just the closure-gain on minimal medium). A degree-preserving shuffle, which permutes metabolite labels across reactant and product slots while keeping each metabolite's in- and out-degree fixed. And a currency-stripped variant, where the standard 50-element cofactor set (ATP/ADP/AMP, NAD(P)(H), FAD, CoA, ACP, water, protons, Pi, PPi, CO2, O2, ammonia, SAM, glutathione, quinones) gets promoted to the food set, so it cannot be the keystone.

Total compute came to about 20 minutes across 12 cores. The code is around 1500 lines including scan drivers, controls, and figure-making.

## What happened

![Figure 1. Closure failure across 777 reconstructions; the phosphorylated tier-1 invariant in panel F.](https://gist.githubusercontent.com/Macaulay001/7dda90c97a96c5613704e8bafc65de55/raw/fig1_closure_failure_v2.png)
*Figure 1. Closure failure across 777 reconstructions; the phosphorylated tier-1 invariant in panel F.*


*Figure 1. Closure failure across 777 reconstructions; the phosphorylated tier-1 invariant in panel F.*


*Figure 1. Closure failure across 777 reconstructions; the phosphorylated tier-1 invariant in panel F.*


Tier 1, first: 773 out of 777 reconstructions across the three pipelines have a phosphorylated top-1 keystone. By pipeline that's 107/108 BiGG, 321/322 EMBL-GEMs, 345/347 AGORA. The per-network base rate of phosphorylation among candidates averages 0.32, so the binomial probability of 773/777 by chance is below 10⁻²⁵⁰. Whatever this is, it isn't noise.

![Figure 2. The phosphate invariant survives across pipelines while the identity shifts.](https://gist.githubusercontent.com/Macaulay001/7dda90c97a96c5613704e8bafc65de55/raw/fig2_tier1_universality_v3.png)
*Figure 2. The phosphate invariant survives across pipelines while the identity shifts.*


*Figure 2. The phosphate invariant survives across pipelines while the identity shifts.*


Here's the core of the computation. The variable names match `src/seed.py`:

```python
# For each candidate, how much does it expand the closure?
X_base, _ = closure_fast(rid_keys, reactants, products, needs, F)
gains = {}
for x in candidates:
    X2, _ = closure_fast(rid_keys, reactants, products,
                         needs, F | {x})
    gains[x] = len(X2) - len(X_base)
keystone = max(gains, key=gains.get)
```

That's it. No training, no ML, no thermodynamics. Just queue-based fixed-point iteration over the reaction network.

The shuffle control is the one I cared about most, because the obvious skeptical reading is "you've rediscovered that hub metabolites are phosphorylated." It isn't that. Top-1 by closure-gain is never top-1 by degree in any of the four organisms I checked closely (in iIT341 the degree king is water, which is obviously not a bootstrap molecule). And when I permuted reactant/product slots to preserve each metabolite's degree exactly while randomizing topology, 12 shuffles across 4 organisms produced zero phosphorylated top-1 keystones. The shuffle top-1 gains were also 4 to 10 times smaller than the real ones. For iML1515 the real NADP gain is 143; the three shuffle gains were 10, 16, 23.

Then the currency control. Promote the standard cofactor set to food so it can't be the keystone, and what emerges is different. Phosphorylated top-1 drops from 21/21 to 3/8 on the organisms I checked. But it isn't random: the new keystones are organism-specific biosynthetic bottlenecks. *E. coli* surfaces the MEP isoprenoid pathway. Human RECON1 surfaces dolichol phosphate. *Mycobacterium tuberculosis* surfaces cobalamin precursors. *Helicobacter* surfaces menaquinone (vitamin K2). Each one is a known lineage-defining piece of biochemistry.

Then strip currency from the network entirely (drop ATP, NAD(P), etc. from every reaction, remove reactions that become empty on either side) and rerun. Phosphorylated top-1 falls to 7/21, which is exactly the base rate of 32%. The phosphate effect vanishes. What appears instead is the anabolic ignition layer: bicarbonate for enterobacteria (feeding into fatty acid synthesis via malonyl-CoA), plastoquinone for cyanobacteria, F420 precursors for methanogens, dehydrodolichol-PP for humans, dUTP for *Thermotoga*. Each one fingerprints the lineage's specialized chemistry.

![Figure 4. The three-tier architecture: universal currency, lineage pathway, anabolic ignition.](https://gist.githubusercontent.com/Macaulay001/7dda90c97a96c5613704e8bafc65de55/raw/fig4_tier_decomposition_v3.png)
*Figure 4. The three-tier architecture: universal currency, lineage pathway, anabolic ignition.*


| Pipeline | n | Phosphorylated top-1 | Modal keystone |
|---|---|---|---|
| BiGG (hand-curated) | 108 | 107 | NADP+ (52%) |
| EMBL-GEMs (CarveMe) | 322 | 321 | ATP (53%) |
| AGORA (microbiome) | 347 | 345 | ATP (55%) |
| Combined | 777 | 773 | — |

The pipelines disagree about *which* phosphorylated molecule wins. BiGG says NADP, the other two say ATP. That disagreement is real and traces to genuine differences in how cofactor turnover is specified across curation conventions. But all three agree that the keystone is phosphorylated. The identity is pipeline-dependent. The phosphorylation is not.

One more check that surprised me. I swept 32 representative organisms across five food regimes including a Miller-Urey-style prebiotic feedstock (water, CO2, ammonia, hydrogen sulfide, methane, HCN, formaldehyde). The phosphorylated-keystone fraction stayed near 24/32 to 26/32 across all regimes, rising to roughly 31/32 after fixing a few BiGG formula-completeness bugs. The bootstrap currency doesn't seem to care whether you feed it glucose or a reducing atmosphere. *E. coli* keeps NADP. Cyanobacteria keep ATP. *Plasmodium*, on reduced food, switches to hemoglobin, which is biologically correct (the parasite digests host hemoglobin for amino acids). The network analysis recovered that parasitic dependency without being told about it.

## What this may mean

The honest version of the claim is layered. There appears to be a universal tier-1 invariant across 777 curated reconstructions: the molecule whose addition most expands network closure carries at least one phosphate atom, in 99.5% of cases. Survives degree-preserving shuffle, survives food-regime sweep, survives swapping the entire reconstruction pipeline. Whatever else this is, it's not a curation artifact.

Below that, tier 2 (currency-promoted) appears to expose lineage-specific cofactor pathways. Heme, cobalamin, menaquinone, dolichol, MEP isoprenoid. Tier 3 (carbon-skeleton only) appears to expose anabolic ignition: bicarbonate, plastoquinone, F420. Each tier may capture a different scale of biochemical inheritance, from the universal cofactor floor down to lineage-specific chemistry.

What I would not claim: that this explains why life chose phosphate, or that it solves abiogenesis, or that it's a biosignature. The empirical fact, on this sample, is that closure-gain on curated metabolic networks consistently picks out a phosphorylated nucleotide-derived molecule. That fact is consistent with Westheimer's chemical argument, but it doesn't prove it. It's possible the curation conventions themselves bias toward phosphorylated cofactors, although the three-pipeline agreement (with very different conventions) makes that explanation harder to sustain.

The tier-2 result is the one I think may matter most practically. Keystone identity at tier 1 clusters cleanly by clade (enterobacteria = NADP, *Plasmodium* = p4a, cyanobacteria = ATP, *Shigella* = Ap5A). It's a quantitative phylogenetic feature derivable in seconds from a reaction network, orthogonal to 16S phylogeny, recovering known clades inside bacteria and eukaryotes. Whether it tells you anything new about the organism, versus just confirming what curators already knew, is the next question.

## Loose ends

A few things still bug me. Greedy seed search isn't provably optimal; the true minimum bootstrap seed could be smaller, and an ILP encoding would settle that. Some BiGG models have bookkeeping reactions for Ap5A hydrolysis that may inflate dinucleoside alarmone rankings (NADP survives this scrutiny, the rank-2 through rank-5 positions might not). And the carbon-skeleton-stripping operation is a blunt instrument. If anyone has a better way to separate "structural role of phosphorylation in carbon flow" from "cofactor turnover effect", I'd like to hear it. The next concrete experiment is the ILP for exact minimum seed; it should run in an afternoon on a single laptop.
