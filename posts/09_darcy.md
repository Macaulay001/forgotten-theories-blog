# D'Arcy Thompson said animal shapes live on a manifold. On 516 mammals, one axis explains 93% of it.

*Part 9 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1917 a Scottish biologist named D'Arcy Wentworth Thompson published a thick book called *On Growth and Form*. The book is famous for one image. Thompson drew the outline of a fish (Argyropelecus) on a square Cartesian grid, then sheared the grid, and the outline of a related fish (Diodon) fell out of the deformation almost for free. He did this for skulls, for crab carapaces, for human pelvises. His suggestion, never quite stated as a theorem, was that related species' body plans differ from one another by simple geometric transformations of a single base form. If true, the phenotypic distance between two cousins should be describable by a handful of numbers, not thousands.

The idea was sidelined for most of the twentieth century. Genetics ate biology, and "shape" felt like a Victorian preoccupation. Geometric morphometrics rehabilitated some of it in the 1990s, but the original quantitative bet, that animal forms sit on a low-dimensional manifold, has not been hammered on as systematically as it might have been.

## What I tried

I wanted a one-evening test of the manifold claim, with public data and nothing fancier than PCA. Real Thompson-style work needs landmark coordinates from digitized specimens, and those live in places like MorphoSource. I deferred that. Instead I went for what I think of as the log-scalar version of his bet: if the manifold is low-dimensional in shape coordinates, then projecting onto a handful of scalar size-and-life-history traits should also collapse onto very few axes.

The dataset I pulled was PanTHERIA, a curated mammal trait database with around 5417 species and 192 measured traits per species. Coverage is patchy, as it always is in ecology. I kept five continuous traits that survive log-transforming and that are widely measured:

- adult body mass (g)
- adult head-body length (mm)
- neonate body mass (g)
- gestation length (days)
- maximum longevity (months)

Then I split by mammalian Order, kept species with all five traits present, log10-transformed everything, mean-centered, and ran PCA via the covariance eigendecomposition. I wanted to see how much variance the first principal component picks up per order, and how many components you need to cross 90%. If Thompson's qualitative intuition has any quantitative teeth, PC1 alone should be a big number.

```python
# keep species with all 5 traits, log-transform, center, eigendecompose
Xlog = np.log10(X)
Xlog = Xlog - Xlog.mean(axis=0)
cov = np.cov(Xlog, rowvar=False)
w, _ = np.linalg.eigh(cov)
w = np.sort(w)[::-1]
cumul = np.cumsum(w / w.sum())
# cumul[0] is PC1's share of variance
```

That is the whole thing. No bootstrap, no permutation null, just the eigenvalues of a 5 by 5 covariance matrix per order.

## What happened

Five orders had at least twenty species with all five traits. Across them, PC1 alone covered between 84% and 96% of total variance. Carnivora and Primates were tightest, Chiroptera the loosest, and the all-orders pool sat at 93%.

| Order        | n   | PC1 var | PC1+2 var | PCs for 90% |
|--------------|----:|--------:|----------:|------------:|
| Carnivora    | 130 | 95%     | 98%       | 1           |
| Primates     | 80  | 96%     | 98%       | 1           |
| Rodentia     | 104 | 93%     | 97%       | 1           |
| Artiodactyla | 84  | 92%     | 97%       | 1           |
| Chiroptera   | 20  | 84%     | 97%       | 2           |
| All combined | 516 | 93%     | 98%       | 1           |

One number to hold onto: across 516 mammal species drawn from five different orders, a single linear axis through the log-trait space accounts for 93% of the variance between species. Two axes get you 98%.

This is not a subtle effect. PC1 is doing essentially all the work. The remaining four components are sharing the last 2 to 7%. Even Chiroptera, the looser case, is at 97% by PC2. If you had to describe the position of a randomly chosen mammal in this five-trait space, you could give one real number and recover almost everything.

I should be honest about what PC1 actually is here. The five traits I picked all scale with body size in mammals. A bigger animal tends to have a longer body, heavier newborns, longer gestation, and a longer life. So the dominant axis I am recovering is, in large part, the famous mammalian size axis. Whales and shrews are at opposite ends of it. That is not a surprise. What is mildly more interesting is that within each Order, the same axis still soaks up 92% or more, which means even after you have already restricted yourself to carnivores or rodents and removed most of the size range, the remaining variation still mostly slides along one direction.

A few things I noticed that I do not want to over-read. Chiroptera (bats) is the only order where one component does not cross 90%. The n is also smallest there (20), so the eigenvalues are noisier. It is also possible that bat life-histories really are more decoupled from body size, given how much flight constrains them, but I cannot tell that from this run. Primates being the tightest order (96% on PC1) surprised me a little. I would have guessed Carnivora, which includes everything from weasels to walruses. The covariance just happens to fall that way on this sample.

The other thing worth saying out loud is that PCA on five log-traits is a generous test. With more traits, especially if some were genuinely orthogonal to size (skull shape ratios, limb proportions, dentition counts), the variance would spread out more. So 93% is not a ceiling on Thompson's claim, it is the answer at the easy end of the difficulty curve.

## What it may mean

Taken at face value, the result lines up with Thompson's intuition. Phenotypic differences between mammal species, at least as measured by these five traits on these 516 species, do appear to live on a low-dimensional manifold. One axis carries most of it, two axes carry essentially all of it. That is consistent with the broader claim that related forms are constrained, that you cannot independently vary every trait, that biology is not free to wander in 192-dimensional space.

What I cannot do from this analysis is claim that Thompson's specific geometric-transformation picture survives. He was talking about coordinate grids and shape deformations, not about correlations between scalar traits. The cleanest test of his original claim would put landmark coordinates from related species through a Procrustes alignment and then PCA, and check whether 2 or 3 components capture most of the shape variance. I have not done that here.

What this run does suggest, with some confidence, is that the size axis in mammals is doing enormous explanatory work. If you wanted a coarse but very efficient code for "where is this mammal in trait space", a single scalar suffices for 93% of the answer. Thompson would probably have nodded. Whether the same compression survives when you move from scalar traits to shape coordinates is the question I would actually like to answer next.

## Loose ends

I would like to repeat this on geometric-morphometric landmark data. MorphoSource has digitized skulls for hundreds of primate, carnivore, and rodent species. A Procrustes plus PCA on that would be the real test of the 1917 claim, and might reveal that once you remove size, the residual shape variance is much higher-dimensional than the scalar trait variance suggests. I would also like to redo the PanTHERIA PCA after regressing out body mass first, to see how much of the 93% is genuinely PC1 doing work versus PC1 just being a relabel of log body mass. That is a couple of evenings of work, not a grant. If anyone has clean landmark data for a single order with 100+ species, please get in touch.
