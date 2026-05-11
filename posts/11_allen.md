# Joel Allen's 1877 rule needs better data than Pantheria can offer. Here's what I found anyway.

*Part 11 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1877, Joel Asaph Allen, working out of the Museum of Comparative Zoology, published a long, discursive paper in the *Radical Review* called "The influence of physical conditions in the genesis of species". Buried inside was a claim that has been quoted in textbooks ever since: warm-blooded animals living in warm climates tend to have longer appendages, while those in cold climates have shorter ones. Longer ears, longer tails, longer limbs all increase surface area and help dump heat. Shorter extremities conserve it. The rule was the logical complement of Bergmann's 1847 observation about body size, and the two together gave 19th-century naturalists a tidy thermal story about the geography of mammal shape.

It is one of those rules that gets repeated more than it gets tested. Symonds and Tattersall did a real comparative test in 2010 using a curated ear-and-tail dataset, but most casual checks rely on whatever morphometrics happen to be lying around. I had Pantheria lying around. So I tried.

## What I tried

Pantheria is a species-level mammal database with about 5417 rows and a few hundred trait columns. It was built in 2009 by Jones and a long list of co-authors, and it has been the workhorse of macroecology for over a decade. What caught my eye were three columns:

- `8-1_AdultForearmLen_mm`
- `13-1_AdultHeadBodyLen_mm`
- `26-4_GR_MidRangeLat_dd` (the midpoint latitude of the species geographic range)

The plan was simple. Take the forearm-to-body-length ratio as a proxy for "appendage size", take absolute latitude as a proxy for cold, and ask whether the ratio drops as you move away from the equator. Allen predicts a negative correlation. If the slope is flat, or worse, positive, that is a problem for Allen, or a problem for the data, and one of the jobs is to figure out which.

I knew going in that Pantheria's coverage of forearm length is patchy. The forearm is the standard morphometric for bats (the bone they actually fly with), so the column is well-populated for Chiroptera and sparse for everyone else. I tried not to let that bias my expectations, but in hindsight I should have looked at the per-order counts before computing anything.

The core of the script is straightforward. Filter to species that have all three numbers, compute the ratio, log-transform it (the ratio is right-skewed), and run Pearson and Spearman against absolute latitude.

```python
valid = [r for r in rows if not (math.isnan(r["forearm"]) or
                                  math.isnan(r["body"]) or
                                  math.isnan(r["lat"]))]
abs_lat = np.array([abs(r["lat"]) for r in valid])
ratio = np.array([r["forearm"] / r["body"] for r in valid])
log_ratio = np.log10(ratio)

r_p, p_p = pearsonr(abs_lat, log_ratio)
rho_s, p_s = spearmanr(abs_lat, log_ratio)
```

That is the whole test. About fifteen lines including the file load. No phylogenetic correction, no body-mass partialing, no climate envelope. A first pass, the kind you run before committing to anything heavier.

## What happened

158 species made the cut. That is the first surprise, sort of. Pantheria has thousands of mammals, but very few of them have all three of forearm, body length, and a range-midpoint latitude on the same row. Most of the missing data is forearm. Out of the 158, 154 are bats.

The overall numbers are not encouraging for Allen.

| Test | Statistic | p-value |
|---|---|---|
| Pearson r(\|lat\|, log(forearm/body)) | +0.109 | 0.17 |
| Spearman ρ | +0.093 | 0.24 |

Allen predicts a negative coefficient. The observed coefficient is weakly positive and not significant. So the headline is "no support", which is fine, except that the sign is wrong, which is interesting.

Restricting to Chiroptera (n=154, which is essentially the whole sample) gives a slightly stronger result in the wrong direction:

| Order | n | Pearson r | p |
|---|---|---|---|
| Chiroptera | 154 | +0.166 | 0.039 |

So within bats, higher-latitude species have *longer* forearms relative to body length, not shorter. The effect is small and the p-value is the kind that would not survive a multiple-comparisons correction, but the sign is the opposite of what Allen would predict.

There are at least two reasons not to take this as a refutation of Allen.

The first is that the forearm of a bat is not really an "appendage" in the heat-dissipation sense Allen had in mind. It is a flight bone. Bat forearm length covaries with wing loading, with foraging mode, with diet, and with all sorts of things that have nothing to do with thermoregulation. There are temperate insectivorous bats with disproportionately long, narrow wings (for fast open-air pursuit) and tropical fruit bats with shorter, broader wings. Wing aerodynamics will swamp any thermal signal in the forearm:body ratio. If anything, the fact that I got a positive correlation at all suggests that the higher-latitude bats in the sample are doing more long-distance commuting flight, not that they are violating Allen.

The second is that the sample is so bat-dominated that calling this a test of Allen's mammal rule is a stretch. Pantheria knows the forearm of one rodent, two carnivores, a handful of marsupials, and 154 bats. There is no order with enough non-Chiroptera coverage to fit a within-order line. The "by order" loop in the script finds exactly one order with n ≥ 10, and that order is Chiroptera.

Allen's original rule was framed for ears, tails, and limbs of mammals broadly, with the canonical examples being the fennec fox versus the arctic fox, or the jackrabbit versus the snowshoe hare. None of those species pairs are testable from Pantheria's forearm column. To test them, you need ear length or tail length at species level, and Pantheria does not carry those at the scale required. Symonds and Tattersall got around this by curating their own dataset, but their data is not openly available, at least not in a form I could pull in an evening.

## What it may mean

I want to be careful here. The cleanest reading of the result is "Allen's Rule for bat forearm length versus latitude is not supported in Pantheria, and the sign is mildly the wrong way". The least-clean reading would be "Allen was wrong". The latter is not what I tested. The forearm:body ratio of bats is a flight-mechanics measurement that happens to be reported in millimeters, and reading it as appendage thermoregulation is a misuse of the column.

If anything, this experiment may be a useful negative example of dataset-driven hypothesis testing. The available column dictates the question, and the question slides quietly away from the original claim. I started out asking about Joel Allen's 1877 thermoregulation rule and ended up asking about wing morphology in temperate-zone bats. Those are not the same question. The answer to the second question may even be interesting (high-latitude bats often *do* have longer, narrower wings, for the reasons above), but it is not Allen's rule.

The honest summary is that Allen's rule has not been tested here. It has been gestured at with the wrong data, and the gesture failed in a way that is more diagnostic of Pantheria than of the rule.

## Loose ends

A real test would use species-level ear length or relative tail length, ideally with phylogenetic generalized least squares to deal with the non-independence of related species, and ideally with mean annual temperature at the range centroid rather than latitude (which conflates temperature with seasonality and continentality). Symonds and Tattersall (2010, *American Naturalist*) did roughly this for rabbits and hares and found Allen-consistent signal. Replicating their approach across more orders would take a few weeks of dataset assembly, mostly hand-curation from museum records, and is probably the right next move. If anyone has a clean ear-or-tail-by-species table covering more than one mammalian order, I would like to see it.
