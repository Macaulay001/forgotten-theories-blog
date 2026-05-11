# I sampled a million random vector pairs in 100 dimensions. Not one violated Cauchy's 1821 inequality, and 99% were within 27 degrees of orthogonal.

*Part 97 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1821 Augustin-Louis Cauchy published *Cours d'Analyse*, a textbook he wrote for his students at the Ecole Polytechnique. Buried in the algebra chapter is the inequality that now bears three names:

    |<u, v>| <= ||u|| * ||v||

Cauchy proved it for finite sums of real numbers. Forty years later Viktor Bunyakovsky extended it to integrals. In 1888 Hermann Schwarz gave the modern proof that works in any inner product space, finite or infinite-dimensional. The Russian textbooks still call it the Cauchy-Bunyakovsky inequality, the German ones Cauchy-Schwarz, and the historical injustice cancels out somewhere in the middle.

I have used this inequality every week for years. The triangle inequality for the L2 norm rests on it. The correlation coefficient is bounded between -1 and 1 because of it. The Heisenberg uncertainty principle is one line of Cauchy-Schwarz away from being trivial. I had never actually drawn a histogram of |<u, v>| / (||u|| * ||v||) for random vectors and watched the bound do its work.

## What I tried

I wanted two things from the simulation. First, the obvious sanity check: take a lot of random vector pairs, compute the normalized inner product c = |<u, v>| / (||u|| * ||v||), and confirm it never exceeds 1. Second, the more interesting question: how does the *distribution* of c depend on the dimension d.

The textbook answer is that for two independent isotropic vectors in R^d, the quantity c follows a known distribution whose variance is 1/d. So as d grows, c concentrates near zero. Two random directions in a high-dimensional space are nearly orthogonal with high probability. That is the seed of half of modern machine learning: random projections work, nearest-neighbour search misbehaves, dot-product attention needs the 1/sqrt(d) scaling factor for a reason.

The plan was a million sample pairs at each of d = 2, 3, 10, 100. Sample u and v as independent standard Gaussians in R^d, compute c, and bin it. Then do a separate d-sweep from d = 2 to d = 1000 to chart the concentration curve and check whether the empirical mean tracks the theoretical sqrt(2 / (pi * d)) for large d.

I also wanted to check the equality case. Cauchy-Schwarz is tight when v is a scalar multiple of u. In floating-point arithmetic, "tight" means "1.0 to within a few ULPs". Worth a sentence.

The whole thing is a few einsum lines. No special libraries.

## What happened

Across all four dimensions, every single one of the 4 million sampled c values landed in [0, 1]. The bound did not fail, which is the most boring confirmation of a 1821 theorem one can imagine, but worth doing once.

Here is the core of the calculation:

```python
for d in [2, 3, 10, 100]:
    U = rng.standard_normal((N, d))
    V = rng.standard_normal((N, d))
    dot = np.einsum("ij,ij->i", U, V)
    nu = np.linalg.norm(U, axis=1)
    nv = np.linalg.norm(V, axis=1)
    c = np.abs(dot) / (nu * nv)
    print(d, c.min(), c.max(), c.mean())
```

That is the entire experiment. The output:

| d   | min c    | max c    | mean c | predicted mean |
|-----|----------|----------|--------|-----------------|
| 2   | 1.58e-06 | 1.000000 | 0.6366 | 2/pi = 0.6366   |
| 3   | 8.01e-07 | 0.999999 | 0.4998 | 1/2             |
| 10  | 2.53e-07 | 0.985052 | 0.2587 | ~ 0.2523        |
| 100 | 5.29e-08 | 0.454959 | 0.0800 | sqrt(2/(100 pi)) = 0.0798 |

The d = 2 case is a clean piece of trigonometry. With u and v isotropic in the plane, the angle between them is uniform on [0, 2 pi], so |cos theta| has expectation 2/pi. The empirical 0.6366 matches to four digits. The d = 3 case is even cleaner: the distribution of c on the sphere in R^3 is exactly uniform on [0, 1], and the empirical mean came out to 0.4998. I find this almost too clean. The geometry of the sphere in three dimensions hands you a uniform distribution on cos theta and there is no integral to do.

The d = 100 panel is the one that matters. Across a million sampled pairs of unit-normalized vectors in R^100, the *largest* normalized inner product was 0.455. Not a single pair had c > 0.5. The bulk of the distribution sits near 0.08 with a width of roughly 0.1. In angular terms, 99% of pairs were within 27 degrees of perpendicular.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/97_cauchy_schwarz_distribution.png)

The lower panel of the figure shows the d-sweep. Both the mean and the 99th percentile of c track the predicted 1/sqrt(d) line over three decades. The theoretical mean sqrt(2 / (pi * d)) sits right under the empirical mean, indistinguishable on a log-log plot.

The equality check: I set v = lambda * u for a thousand random vectors in R^7 and random scalars lambda. The maximum deviation of c from exactly 1 was 4.44e-16, which is one unit in the last place at double precision. Cauchy-Schwarz becomes tight precisely when u and v are linearly dependent, and floating-point arithmetic respects that to round-off.

## What it may mean

The numerical check itself proves nothing the theorem did not already prove. What the simulation makes visible is the geometry of high dimensions, and the inequality is what makes that geometry tractable.

The fact that two random vectors in R^100 are nearly orthogonal is the load-bearing fact behind several modern tools. Random projections (Johnson-Lindenstrauss) work because a random orthogonal-ish projection preserves pairwise distances up to a factor that depends on 1/sqrt(d). Locality-sensitive hashing exploits the same concentration: random hyperplanes split high-dimensional points in a way that respects cosine similarity. The scaled dot-product attention in transformers divides by sqrt(d) for the same reason: without that scaling, the softmax of inner products in R^512 would saturate to a one-hot vector almost every time.

There is also a lesson about correlation. The correlation coefficient between two n-sample variables is the normalized inner product of their centered vectors in R^n. Cauchy-Schwarz is what bounds it to [-1, 1]. The high-dimensional concentration says that if you correlate two random length-1000 series, you should expect |r| ~ 1/sqrt(1000) ~ 0.03. Anything much larger needs an explanation, and the size of the expected fluctuation is itself a Cauchy-Schwarz consequence rather than a separate fact.

One more way to put it. The inequality has a slick proof in two lines: expand ||u - t v||^2 >= 0 as a quadratic in t, demand the discriminant to be non-positive, and you are done. The proof never mentions dimension. The geometric content I have been describing, the concentration near orthogonality, comes from the *measure* you put on pairs, not from the inequality itself. The 1821 algebra is a hard ceiling. The histograms are how often you bump into it.

None of this is contested. The inequality is a theorem and the consequences are textbook. What I had not done was watch the histograms collapse with d in front of me. The picture is sharper than the abstract statement.

## Loose ends

The simulation used isotropic Gaussians. The bound c <= 1 is universal but the *shape* of the distribution depends on the sampling measure. Sparse vectors, heavy-tailed distributions, or vectors constrained to the simplex have noticeably different distributions of c, even though the bound still holds. A nice follow-up would be to check what happens for binary {0, 1} vectors with density p, which is the regime that matters for set-similarity and Jaccard-style metrics. That is an afternoon of work. The integral version, Bunyakovsky's contribution, would need a different setup with random functions on [0, 1], which would mostly be an exercise in choosing a basis.
