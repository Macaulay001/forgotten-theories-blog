# Cope said animals get bigger over time. On 500 Pleistocene mammals, the data fail him, but for the wrong reason.

*Part 7 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Edward Drinker Cope was, among other things, a paleontologist who feuded with Othniel Marsh over dinosaur bones in the American West. In 1887, in *Origin of the Fittest*, he stated what became one of the more recognizable empirical claims in paleontology: animal lineages tend to evolve toward larger body size over geological time. Cope's mechanism was selectionist. Bigger animals were thought to win in competition, in predation, in thermoregulation. The idea was mainstream for most of the 20th century. Then it quietly faded. Stanley (1973) showed many invertebrate lineages stay flat or even shrink. Jablonski (1997) reported mixed support. The Heim et al. (2015) *Science* paper partly rehabilitated Cope for marine animals over 500 million years, but the original universal version has never had a clean modern test on a phylogenetically corrected panel.

So I wanted to try the cheapest possible version of the test, just to see what shakes out.

## What I tried

The most accessible curated body-mass dataset I know of is Smith et al. (2003), the MOMv3.3 file, sometimes called the "Body Mass of Late Quaternary Mammals" compilation. It is a tab-separated dump with about 5,700 rows, each one a mammal species or subspecies. Each row carries a continent, an extant-or-extinct status flag, taxonomy down to species, and a body-mass estimate in grams (also given as log10). The "extinct" flag in MOM means extinct in the late Quaternary, roughly the last 125,000 years. That sweeps in the Pleistocene megafauna: mammoths, mastodons, giant ground sloths, the Australian Diprotodontidae, the various American camels and horses, dire wolves, short-faced bears.

That last fact should already make a careful person nervous, but I'll come back to it.

My plan was small. Pull the file. Parse the records. Split by status. Compare the log10 body mass of extant mammals to the log10 body mass of recently extinct mammals using Mann-Whitney. If Cope is right in the most naive way, the living mammals should be larger on average than the ones we just lost. If Stanley's passive-trend version is more right, the lower bound stays put and the difference is smaller or zero.

There's also a family-level test that I think is closer to what Cope was actually claiming. Within a single mammal family that has both living and recently extinct members, who is bigger? That mostly factors out the "mammals as a clade are huge" baseline and asks the within-lineage question. I required at least three extant and three extinct species per family, ran a two-sided Mann-Whitney inside each family, and counted how many families went each direction.

The whole thing is a couple of hundred lines, the core of which is the per-family loop:

```python
for fam, d in by_fam.items():
    if len(d["extant"]) < 3 or len(d["extinct"]) < 3:
        continue
    u, p = mannwhitneyu(d["extant"], d["extinct"], alternative="two-sided")
    fam_stats.append({
        "family": fam,
        "median_extant": st.median(d["extant"]),
        "median_extinct": st.median(d["extinct"]),
        "delta": st.median(d["extant"]) - st.median(d["extinct"]),
        "p": p,
    })
```

That's it. No phylogenetic correction, no age binning, no body-mass uncertainty propagation. The crudest possible Cope test on the cleanest available modern mammal dataset.

## What happened

The headline result is not subtle. Median log10(mass, g) for extant mammals in MOM is 1.92, which is roughly 83 grams. That's reasonable. Mammals are mostly rodents and shrews and bats, and once you weight by species count rather than biomass, the median mammal is a thing that fits in your hand. Median log10(mass, g) for the extinct Pleistocene mammals in the same dataset is 5.26, or about 180 kilograms. The Mann-Whitney p-value for "extant smaller than extinct" comes out below 10^-9. The naive Cope prediction, that today's mammals are larger than yesterday's, is refuted, and refuted hard, and in the opposite direction.

| group | n | median log10(mass, g) | approx. mass |
|---|---|---|---|
| extant | ~5,300 | 1.92 | 83 g |
| extinct (Pleistocene) | ~190 | 5.26 | 180 kg |

The family-level numbers are more interesting and, in a way, more honest. Of the families with at least three living and three Pleistocene-extinct species, roughly the same fraction go each way once you look at the median shift, and very few of the per-family tests survive p < 0.05 in either direction. The big global signal is being driven not by all families drifting smaller but by a small number of families that lost their largest members and are now represented in the modern fauna only by their smaller relatives. Equidae, Camelidae, Megatheriidae, the various proboscideans. The within-family deltas are negative because that family's largest species died in the Pleistocene and only the smaller cousins, or the smaller relatives in other continents, are still around.

This is the moment where the test stops being about Cope and starts being about extinction selectivity.

The MOMv3.3 panel is not a fair window into evolutionary trends. It is a snapshot of the last interglacial transition, during which something, climate or humans or both, preferentially removed large-bodied mammals. Of the 24 fully extinct Pleistocene mammals I looked at most closely, every single one is a megafaunal species. There is no "extinct shrew" in this dataset. There can't be, because the dataset was built from a fossil and subfossil record that disproportionately preserves and reports large bones, and from extinction events that disproportionately took large animals. The thing I am measuring when I say "extant is smaller than extinct on this panel" is the Pleistocene megafauna extinction filter. It is real, it is statistically overwhelming, and it has almost nothing to do with what Cope was claiming about within-lineage evolutionary change over millions of years.

So the answer to "does the naive Cope claim survive the MOM dataset" is no, it does not, the p-value is comically small. But the answer to "did we actually test Cope's rule" is also no.

## What it may mean

If I'm being honest about what this short experiment shows, it's mostly a lesson about dataset shape. The cleanest available compiled mammal body-mass dataset can't test Cope's rule on its own, because the extinction filter at the dataset's temporal edge is sharper than any evolutionary signal inside the panel. You can refute the naive form of the claim, but you do it for a reason Cope would not have considered a fair counterexample. He was talking about lineages evolving toward larger size over deep time. He was not talking about which size class gets killed off in the last ten thousand years.

It also suggests, gently, that the modern fauna we see is not a random sample of the recent past. If you tried to fit any trend model to mammalian body size using only living species, you would be working with a distribution that has had its right tail clipped within the last hundred thousand years. Any "mammals are getting smaller" or "mammals are getting larger" claim that uses only extant data is going to be biased in a direction set by Pleistocene extinction selectivity. That seems worth keeping in mind.

The Heim et al. (2015) result for marine animals, with phylogenetic correction over half a billion years, is closer to a real test, and they did find a Cope-consistent signal in marine genera. I am not equipped, in a short notebook session, to do the equivalent thing for mammals across the Cenozoic.

## Loose ends

A proper test would need within-lineage comparisons across deep time. Ancestral horse body mass against descendant horse body mass, for many lineages, with branch-length-aware comparative methods. The data exist in the Paleobiology Database measurement tables, but pulling them, cleaning the body-size estimates, and matching to a phylogeny is closer to a week of work than an afternoon. If anyone reading this has already done that pull for mammals, I would like to see it. Otherwise the next step would be to start with PBDB occurrences for a handful of well-sampled mammalian families and check the within-family trend over 30 to 50 million years.
