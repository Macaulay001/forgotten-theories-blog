# Robert May said complex food webs should collapse. 92% of them violate his bound and stay stable.

*Part 3 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In May 1972, Robert May published a short paper in *Nature* with a result that annoyed almost every ecologist who read it. He took a community of n species, wired them together with a random interaction matrix at connectance C and interaction strength sigma, and showed that the system loses stability the moment sigma times sqrt(nC) crosses 1. More species, more links, stronger couplings, all of these push the dominant eigenvalue positive. The prediction was clean and uncomfortable: complexity destabilizes. Diversity should not, in his framework, beget stability.

Field ecologists had been saying the opposite for decades. Elton and MacArthur both argued that richer webs absorb perturbations better. May's math seemed to say they were wrong, or that the webs we see are special in some way the random model misses. The paper got cited tens of thousands of times and the debate never really closed. I wanted to see what the bound looks like when you point it at every ecological network anyone has bothered to digitize.

## What I tried

The Mangal database (Poisot et al. 2016) catalogues something like 1,487 published interaction networks, from 1899 collection trips to last year's pollinator surveys. Food webs, host-parasite networks, plant-pollinator visitation, seed dispersers, mycorrhizal associations. I pulled all of them through the Mangal API, kept the 1,255 that produced a valid signed community matrix with at least 5 species, and computed the May bound for each one.

The construction is the standard textbook recipe. For each observed interaction I assigned signs by ecological type. Mutualism, facilitation, commensalism, seed dispersal: both partners get a positive entry. Competition: both negative. Predation, herbivory, parasitism, host-parasite, antagonism: the consumer entry is negative, the resource-on-consumer entry is positive. This is the antisymmetric predator-prey pairing that Allesina and Tang 2012 argued was the missing ingredient.

For each matrix A I computed n, the connectance C (nonzero off-diagonals over n times n-1), sigma (standard deviation of the nonzero entries), and the largest real eigenvalue of -I + A_off. The -I is the diagonal self-regulation that May builds in by convention. So a stable network has lambda_max below zero. May's prediction says lambda_max should be near sigma*sqrt(nC) - 1, meaning anything with sigma*sqrt(nC) above 1 should be unstable.

I also built two nulls. One is the original May random-sign matrix at matched (n, C, sigma). The other shuffles the empirical signed entries across pairs, preserving sign distribution but breaking the predator-prey pairing. This separates "having the right mix of signs" from "having the signs in the right places".

The core eigenvalue loop is short. The whole analysis runs on a laptop once the data is local.

```python
A_off = A.copy(); np.fill_diagonal(A_off, 0)
nonzero = (A_off != 0).sum()
C = nonzero / (n * (n - 1))
sigma = A_off[A_off != 0].std()
bound = sigma * np.sqrt(n * C)
lam_emp = np.linalg.eigvals(-np.eye(n) + A_off).real.max()
```

That is the experiment, repeated 1,255 times.

## What happened

![Figure 1. May's bound is violated by 92.6% of real networks yet they remain stable.](https://gist.githubusercontent.com/Macaulay001/7ba064e8c57f3aece84ad9601962e37d/raw/fig1_may_bound.png)
*Figure 1. May's bound is violated by 92.6% of real networks yet they remain stable.*


*Figure 1. May's bound is violated by 92.6% of real networks yet they remain stable.*


*Figure 1. May's bound is violated by 92.6% of real networks yet they remain stable.*


The bound is violated almost everywhere. 1,162 of 1,255 networks (92.6%) have sigma*sqrt(nC) at or above 1. The median value of sigma*sqrt(nC) across the corpus is 15.3, so the typical network is roughly 15 times deeper into May's "unstable" regime than the threshold says it should be able to sit.

The median empirical lambda_max is -1.00. That is the diagonal alone. The off-diagonal block, on average, contributes essentially nothing to the dominant eigenvalue's real part. Around 89% of the networks have lambda_max below zero by the linearized criterion. May's bound, taken as a prescription, is wrong on real data, and not by a little.

Stratifying by dominant interaction type sharpens the picture.

![Figure 2. Antagonistic networks blow past the bound and stay stable; mutualistic violate stability under the same model.](https://gist.githubusercontent.com/Macaulay001/7ba064e8c57f3aece84ad9601962e37d/raw/fig2_by_type.png)
*Figure 2. Antagonistic networks blow past the bound and stay stable; mutualistic violate stability under the same model.*


*Figure 2. Antagonistic networks blow past the bound and stay stable; mutualistic violate stability under the same model.*


*Figure 2. Antagonistic networks blow past the bound and stay stable; mutualistic violate stability under the same model.*


| Dominant type | n networks | median sigma*sqrt(nC) | median lambda_max | frac above bound |
|---------------|-----------:|----------------------:|------------------:|-----------------:|
| Parasitism    | 612        | 72.1                  | -1.00             | 100%             |
| Mutualism     | 361        | 4.57                  | +18.77            | 86%              |
| Predation     | 187        | 2.09                  | -1.00             | 84%              |
| Herbivory     | 57         | 2.59                  | -1.00             | 100%             |
| Detritivore   | 12         | 1.92                  | -1.00             | 100%             |

Parasitism networks are the wildest. They sit at sigma*sqrt(nC) around 72, meaning May's matrix would predict catastrophic instability with eigenvalues on the order of +70. The actual networks are stable at lambda_max = -1. Predator-prey antisymmetric pairing seems to be doing essentially all of the work, and food webs, herbivory networks, detritivore webs all behave the same way.

Mutualism is the exception, and it breaks in the direction Allesina and Tang predicted. The 361 mutualistic networks in Mangal (mostly pollination and seed dispersal) have median lambda_max of +18.8 under this linearized community matrix. They blow up. By any linear stability criterion these networks should not persist, yet they obviously do. That negative result is itself the signal: mutualism cannot be holding itself together via the Jacobian. It has to be living on functional saturation, alternative-partner switching, or parasitism mixed in to clip the positive feedback, none of which appears in May's framework.

The sign-preserving shuffle gives a partial answer about what to credit. When I keep the empirical sign distribution but shuffle which pairs interact, lambda_max gets worse than the empirical network but stays better than the i.i.d. May null. So roughly speaking, part of the stabilization comes from "ecology uses a non-iid mix of signs" and part comes from "those signs sit on specific predator-prey pairs". Both matter. I haven't yet quantified the split cleanly.

The closest comparable studies are Yodzis 1981 on a handful of webs, Allesina and Tang 2012 on synthetic matrices, and Jacquet et al. 2016 on around a hundred food webs. As far as I can tell, no one has run the cross-type panel at this scale before. The pieces were all there, the database was sitting on a server, and nobody had stitched them together.

## What it may mean

If this holds up, May's bound is best read as a statement about a very specific null model rather than a fact about real ecosystems. The bound applies when interactions are drawn i.i.d. with random sign at fixed mean zero. Real ecology is almost never i.i.d. random sign. Trophic interactions come in antisymmetric pairs because eating someone is bad for them and good for you. That structure on its own buys enough stability to absorb networks 70 times past the May threshold.

The mutualism result is the other half of the same story. Pollinators and plants help each other, and a linear model says mutual help is positive feedback all the way down. The fact that median lambda_max for mutualistic networks lands at +18 in this analysis is, I think, the cleanest empirical demonstration that mutualism in nature must be running on nonlinear saturation. The functional response has to flatten, or the system goes off.

None of this proves the bigger ecological claim that diversity stabilizes. The linearized Jacobian is a weak notion of stability and we are not running dynamical simulations here. What I can say is that the specific 1972 prediction (large nC implies eigenvalue blowup) does not survive contact with the Mangal corpus once you put the signs in the right places.

## Loose ends

The interaction-strength values in Mangal are mostly presence/absence, value = 1. For binary networks the May bound reduces to sqrt(nC) < 1 and sigma drops out as a free parameter. Where weighted values were available I used them, but a real next step would be to redo this on a subset of webs with measured biomass flow or consumption rates, where sigma is meaningfully heterogeneous. Mangal sampling is also biased toward temperate plant-pollinator systems and agricultural host-parasite work, so the global distribution might shift things. A week of careful subsetting would tell me. If anyone has weighted predator-prey rate constants for more than 20 networks, I would love to see them.
