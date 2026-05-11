# Pyotr Kropotkin claimed cooperation builds richer ecosystems. 1,217 food webs may agree.

*Part 6 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Pyotr Kropotkin was a Russian prince who renounced his title, escaped a tsarist prison, and spent the rest of his life arguing that biologists had read Darwin too narrowly. In 1902 he published *Mutual Aid: A Factor of Evolution*, a book stitched together from observations he made as a young geographer in Siberia. He had gone east expecting to see the gladiatorial nature that T. H. Huxley described in his essays. Instead he saw horses huddling against blizzards, cranes posting sentries, and beetles burying carrion in pairs. His claim was simple and, for the time, almost heretical: cooperation is at least as important an evolutionary factor as competition, and the species that practice it should be more successful.

The book was admired and then quietly shelved. Huxley's framing of nature as a war of each against all was easier to teach, and the genetics revolution that followed had no patience for arguments built on field anecdotes. Kropotkin's specific empirical claim, that cooperative species sustain richer communities, has never really been tested at scale.

## What I tried

I wanted a check that did not rest on reading the book sympathetically. The cleanest path I could see was through ecological network data. If Kropotkin is right in any measurable way, then communities that are mostly held together by mutualistic interactions, such as pollination, seed dispersal, and facilitation, should look different from communities that are mostly held together by predation, parasitism, and herbivory.

I already had the Mangal interaction database sitting in the theory 03 folder from an earlier May-stability check. Mangal is a curated dump of published ecological networks, mostly bipartite plant-pollinator graphs and tripartite food webs, going back roughly a century of field work. After dropping networks with missing interaction labels I had 1,255 valid networks, and 1,217 of them were dominantly mutualistic or dominantly antagonistic. The rest were genuinely mixed and got set aside.

The classification rule was crude on purpose. For each network I looked at the published interaction type for each edge. If at least 70% of the edges were labelled as mutualism, facilitation, commensalism, or seed dispersal, I called the network mutualistic. If at least 70% were labelled predation, herbivory, parasitism, parasitoidism, or host-parasite, I called it antagonistic. Anything in between stayed "mixed" and was dropped from the headline comparison.

For each network I then pulled two summary numbers that theory 03 had already computed. The first is species count, just the number of distinct nodes. The second is connectance, the fraction of all possible directed edges that are actually present. Connectance is the standard density measure for ecological networks and it shows up in every May-stability paper from the 1970s onward, so I trusted it.

The plan was as compact as it sounds: bucket the networks, compare the medians, run a Mann-Whitney U on the species counts to be sure I wasn't being fooled by the means, and write down what I saw. The whole pipeline runs in under a minute on a laptop.

```python
for r in rows:
    t = r.get("dominant_type", "")
    if t in ("mutualism", "mutualistic"):
        by_type["mutualistic"].append(r)
    elif t in ("predation", "parasitism", "herbivory",
               "predator-prey", "host-parasite"):
        by_type["antagonistic"].append(r)

ns_mut = [r["n"] for r in by_type["mutualistic"]]
ns_ant = [r["n"] for r in by_type["antagonistic"]]
u, p = mannwhitneyu(ns_mut, ns_ant, alternative="greater")
```

## What happened

The split was lopsided in a way I had not expected. Out of 1,217 dominantly-classified networks, 856 were antagonistic and only 361 were mutualistic. Ecologists publish a lot more food webs than pollination webs, which probably says more about funding and field method than about how often these interaction types occur in nature.

The size difference was large.

| Class         | n networks | median species | median connectance |
|---------------|-----------:|---------------:|-------------------:|
| Mutualistic   | 361        | 30             | 0.088              |
| Antagonistic  | 856        | 12             | 0.222              |

The median mutualistic community has roughly 2.5 times as many species as the median antagonistic one. The Mann-Whitney U test on the alternative "mutualistic species count is greater" returns p < 1e-57, which is comfortably past any reasonable threshold. I checked the histograms by hand because numbers that small make me suspicious, and the distributions really are shifted, not just dragged around by a few outliers. The mutualistic distribution has a long tail past 100 species. The antagonistic distribution piles up between 6 and 20 species and then thins out fast.

Connectance moves in the opposite direction, which is interesting on its own. Antagonistic networks are denser, with about 22% of possible edges realised in the median case, while mutualistic networks sit closer to 9%. A larger, sparser graph is a more modular graph, and modularity is the kind of structural property that ecologists since Robert May have linked to community persistence. So on two counts the mutualistic networks look like the kind of system Kropotkin would have pointed at: more species, more loosely connected, with fewer of the dense feedback loops that destabilise toy ecosystems.

I want to be careful here. The networks are not directly comparable. A plant-pollinator network is almost always bipartite by construction, with dozens of plants on one side and dozens of pollinators on the other, while a food web is usually a few trophic layers with a small number of top predators. The structural difference may explain part of the species-count gap before any ecology gets involved. I tried to control for this by restricting to networks with at least three trophic-style levels on the antagonistic side, and the gap shrank from 2.5x to about 1.9x but did not close. That feels like real signal sitting under a structural confound, although I would not bet the farm on the exact factor.

The connectance result is harder to wave away. Even if mutualistic networks are bipartite for structural reasons, there is no obvious reason that bipartiteness would push connectance from 22% down to 9%. The two numbers seem to be saying that cooperative communities really are spread out across more partners with thinner per-pair coupling, which is qualitatively what Kropotkin described when he wrote about herds and flocks providing many small acts of help rather than dense exchanges between a few individuals.

## What it may mean

If you want to read Kropotkin generously, the direction of this result is the one he predicted. Communities where cooperation dominates are larger, in the sense of sustaining more species per network, and they are structured in a way that classical theoretical ecology associates with persistence. The pattern is consistent across a thousand-plus independent published networks, which is more evidence than Kropotkin ever had.

If you want to read him strictly, the result is correlation and not much more. Plant-pollinator systems involve many plant species crossed with many animal partners almost as a definitional matter. Food webs are constrained by the trophic pyramid: you cannot fit a hundred top predators on top of fifty herbivores. Some of the species-count gap comes from those structural facts and not from any causal contribution of cooperation. The arrow of causation is exactly what this kind of test cannot pin down.

What this analysis does buy is a softer version of Kropotkin's claim. Mutual aid is at minimum compatible with rich, persistent communities, and it appears in the data in places where competition does not produce the same diversity. That is a long way from his stronger evolutionary argument, but it is not nothing, and it is the first time I have seen the prediction checked on more than a handful of systems.

## Loose ends

A within-clade test would be much stronger. Cooperative-breeding birds versus solitary-breeding birds in the same family, paired with BirdLife range sizes, would isolate the cooperation signal from the structural confound. I have the AviList taxonomy and a partial Cockburn 2006 list of cooperative breeders ready to merge, but the GBIF range pull is slow and I have not done it yet. If anyone has cleaned cooperative-breeding labels for more than 200 bird species, please get in touch. The next experiment is probably a weekend, not a grant.
