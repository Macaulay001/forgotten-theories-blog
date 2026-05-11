# Joseph Greenberg's 30 languages were too few. WALS has 3,573, and his universals start to wobble.

*Part 2 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1963, Joseph Greenberg published a paper that quietly reshaped how linguists thought about grammar. He looked at 30 languages he could get good descriptions of (English, Turkish, Swahili, Japanese, Maori, Hindi, a Quechuan language, a handful of others) and proposed 45 implicational universals about word order. Things like: if a language puts the verb first (VSO), it will also be prepositional. If adjectives come before nouns, demonstratives will too. The claims were sharp, and many of them held at 100% across his sample.

![Figure 2. Greenberg's 30-language sample vs WALS — rates collapse on bigger sample.](https://gist.githubusercontent.com/Macaulay001/16e9ebc0e655ad6c23bdd64afde3be9c/raw/fig2_g30_vs_wals.png)
*Figure 2. Greenberg's 30-language sample vs WALS — rates collapse on bigger sample.*


*Figure 2. Greenberg's 30-language sample vs WALS — rates collapse on bigger sample.*


Most of them stayed canonical for decades. They show up in undergraduate syntax classes. They have feature codes in typology databases. Some of them have been gently contested (Dryer 1992, Dunn et al. 2011), but the broad picture, that there really are deep cross-linguistic constraints on word order, has mostly survived. What I wanted to know was simpler. Greenberg looked at 30 languages. We now have catalogues with thousands. What happens if you run his implications against everything?

## What I tried

I pulled the WALS dataset (World Atlas of Language Structures, the 2014 CLDF release). It has 3,573 languages and 192 typological features, with roughly 76,000 filled-in value cells across the matrix. Coverage is patchy. For most languages you get maybe 30 to 50 features, not all 192. But the word-order features Greenberg cared about (basic constituent order, adposition order, genitive order, adjective order, demonstrative order, numeral order, relative-clause order) are among the better-covered cells, with several hundred to over a thousand languages each.

The plan was to encode each universal as a logical implication on the WALS feature codes and just tabulate. Greenberg's Universal 3, for instance, says: if a language is VSO, it is prepositional. In WALS that becomes: feature 81A equals 3 implies feature 85A equals 2. Languages where the antecedent doesn't apply get skipped, languages where it does get counted as confirmations or violations, and the confirmation rate falls out.

The trick is getting the implications right. Some of Greenberg's 45 use categories WALS doesn't carry at sufficient granularity (question-particle position, certain agreement patterns, some morphology). Some get paraphrased in WALS in ways that subtly change the logical content. I ended up with 13 implications that I felt comfortable encoding without bending the original claim. That includes his classics (U1, U2a, U2b, U3, U4, U17, U18a, U18b, U24), a strong-form combined version of U3, and Matthew Dryer's OV→Postp and VO→Prep correlations as a cross-check.

Each predicate looks like this in practice:

```python
def universal_17(feats):
    # VSO → adjective after noun
    v81 = feats.get("81A")
    v87 = feats.get("87A")
    if v81 != 3 or v87 not in (1, 2):
        return ("skip", None)
    return (v87 == 2, "VSO → N-Adj" if v87 == 2 else "VSO but Adj-N")
```

Skip rules matter. I dropped "no dominant order" cases rather than counting them, which is the same convention Greenberg used. Then I also re-ran the same predicates restricted to Greenberg's original 30-language list, to see what his sample would have told him.

## What happened

![Figure 1. Confirmation rates on WALS for the 13 universals tested.](https://gist.githubusercontent.com/Macaulay001/16e9ebc0e655ad6c23bdd64afde3be9c/raw/fig1_rates.png)
*Figure 1. Confirmation rates on WALS for the 13 universals tested.*


*Figure 1. Confirmation rates on WALS for the 13 universals tested.*


*Figure 1. Confirmation rates on WALS for the 13 universals tested.*


None of the 13 universals collapsed. All 13 confirmed at 75% or higher on the full WALS sample. So Greenberg was directionally right on every implication we could test. That is a real finding, and it is the kind of thing that gets quietly lost when people foreground the exceptions.

But the absoluteness collapsed almost everywhere. On his 30 languages, 12 of the 13 implications confirmed at exactly 100%. On WALS 3,573, only one (U24, relative-noun implies genitive-noun) approached that, at 98.5% with 2 exceptions globally. The rest came down, sometimes a little, sometimes a lot.

| Universal | Greenberg 30 | WALS 3,573 | shift |
|---|---:|---:|---:|
| U17 (VSO → NAdj) | 100% (6/6) | 75.3% (61/81) | −25 |
| U2a (Prep → N-Gen) | 100% (13/13) | 86.7% (351/405) | −13 |
| U18b (AdjN → NumN) | 100% (10/10) | 87.2% (251/288) | −13 |
| U18a (AdjN → DemN) | 100% (10/10) | 91.4% (266/291) | −9 |
| U3 (VSO → Prep) | 100% (6/6) | 92.7% (76/82) | −7 |
| U1 (S before O) | 100% (25/25) | 96.6% (1147/1187) | −3 |

The U17 number is the one that surprised me. On Greenberg's 6 VSO languages, every single one put the adjective after the noun, so the implication looked airtight. On the 81 VSO languages in WALS with an adjective-order coding, 20 of them violate it. That is a quarter of the sample. The exceptions are clustered in a specific way: Salishan (Pacific Northwest of North America), Mayan, Tsimshianic, and parts of Austronesia. These are exactly the VSO families that Greenberg's 30-language sample under-represents. He had Maasai and Berber for VSO. He didn't have, say, Squamish.

A different kind of pattern shows up in U2. Greenberg implied symmetry between his Universal 2a (prepositional implies noun-genitive) and 2b (postpositional implies genitive-noun). WALS pulls them apart. Postp → Gen-N confirms at 97.1% (442 of 455 languages). Prep → N-Gen confirms at 86.7%. The 54 prepositional-but-Gen-N exceptions are dominated by Austronesian (22 languages) with smaller contributions from Indo-European and North Halmaheran. So one direction of the implication is nearly absolute and the other is family-conditioned.

I ran a drop-top-family ablation to make this quantitative. For each universal, I removed the largest exception family from the sample and recomputed. U18b (adjective-noun implies numeral-noun) gains 3.4 percentage points when Sino-Tibetan is removed, and the Sino-Tibetan exceptions cluster geographically about 6× tighter than a random subsample. That is roughly the pattern Dunn et al. (2011, *Nature*) described when they argued for "lineage-specific trends" rather than universal laws. On this dataset it appears to apply to U18b and U18a, but it does not apply to U1, U2b, U4, or U24, which stay near-universal even when you knock out the top exception family.

So the picture I came away with is messier than "Greenberg was right" and messier than "Greenberg was wrong". On 13 implications, 5 are near-absolute on the full sample (U1, U2b, U4, U24, OV→Postp), 3 are strong tendencies in the low 90s, and 3 are moderate tendencies between 75% and 90% that may partly be areal phenomena rather than universal constraints.

## What it may mean

If this holds up, what it suggests is that Greenberg's 30-language sample was good enough to spot the directional patterns but too small to estimate how absolute they were. Twelve of thirteen implications at 100% is not really a finding about the world. It is a finding about the sample. With 30 languages and an implication where the true rate is 92%, the chance of seeing zero violations is roughly 8%, so getting several such universals all at 100% is plausible if his languages happened to be the "clean" ones. Which, given how he picked them (well-documented, mostly from a handful of families he knew), they probably were.

The other thing this may say is that some of what we call universals are partly areal. U18b's Sino-Tibetan clustering looks like an East/Southeast Asian linguistic-area pattern (Sino-Tibetan, Hmong-Mien, Tai-Kadai) more than a constraint of human language. That is not a proof, just a strong hint, and the right follow-up is a phylogenetic-comparative analysis the way Dunn et al. did it, not a count of family labels.

I don't want to oversell this. None of the 13 universals were refuted, the directional core of Greenberg's claim survives, and the OV/VO correlations from Dryer's 1992 reformulation look fine on the full sample (97% and 92%). The thing that erodes is the rhetoric of absoluteness, not the underlying tendency.

## Loose ends

I tested 13 of Greenberg's 45. The other 32 involve question-particle position, anaphoric reference, agreement, and morphology where WALS either doesn't code at the right granularity or paraphrases in ways that change the implication. Getting at those would mean either Grambank (which has finer coding) or hand-curating from grammars, both of which are weeks of work. The family-clustering analysis I did uses WALS family labels, which are coarse (Austronesian is enormous); a genus-level or phylogenetic analysis would refine the lineage-specificity story. The whole thing took about a day of compute and is small enough that anyone can rerun it from the WALS CLDF dump in an afternoon.
