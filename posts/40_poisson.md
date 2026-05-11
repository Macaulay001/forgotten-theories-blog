# I refit Bortkiewicz's 1898 Prussian horse-kick deaths against Poisson's 1837 law. The textbook example still holds at chi-square p = 0.38.

*Part 40 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1837 Siméon Denis Poisson wrote a 415-page book on the probability of court verdicts. Somewhere in the middle of it he derived a limit theorem for the binomial distribution: if you have many independent trials, each with a tiny chance of success, the count of successes converges to a clean little formula, P(k) = exp(-λ) λ^k / k!. The result sat on the shelf as a mathematician's curiosity for sixty-one years. Nobody had a good dataset for it.

Then in 1898 Ladislaus von Bortkiewicz, a Polish-Russian statistician working in Berlin, found one. He had been reading Prussian military mortality tables and noticed an oddly specific cause of death: kicked by a horse. Across 14 cavalry corps, recorded for 20 years, that gave 280 corps-years of data with a mean of 0.7 deaths per corps-year. Bortkiewicz compared the empirical distribution to Poisson's formula and found a match so clean it became the textbook illustration for the next 128 years.

I had quoted Bortkiewicz in lectures without ever actually refitting his numbers. So I spent an afternoon doing it.

## What I tried

The Bortkiewicz dataset is five rows. Of 280 corps-years:

| deaths k | corps-years |
|---:|---:|
| 0 | 144 |
| 1 | 91 |
| 2 | 32 |
| 3 | 11 |
| 4 | 2 |

Total deaths: 196. Mean per corps-year: 196/280 = 0.7. That gives λ̂ = 0.7 by maximum likelihood, since the MLE for a Poisson is just the sample mean. Expected counts under Poisson(0.7) are 139.0, 97.3, 34.1, 8.0, 1.6. Then a chi-square against the observed row, with two cells merged at the tail because expected fell below 5, and one degree of freedom subtracted for the estimated λ.

I wanted three more datasets to keep the test honest. A theory that fits one famous old example can fail on the next thing you throw at it.

First, a synthetic Geiger-counter panel. Rutherford and Geiger in 1910 counted alpha particles emitted by polonium in 2608 intervals of 7.5 seconds, and reported a near-perfect Poisson fit with λ around 3.87. I cannot rerun their experiment, but I can simulate it under true independence and see whether the test recovers the right answer. Sample 2608 draws from Poisson(3.87), tabulate, refit.

Second, a homogeneous rare-disease panel. 500 region-years, every region with the same per-year incidence λ = 1.2. This should pass Poisson trivially. It is the control.

Third, a heterogeneous rare-disease panel. Same 500 region-years, but the per-region λ is itself drawn from a Gamma(shape=2, scale=0.6) distribution. Mean exposure is still 1.2, but exposure varies. This is what real disease counts look like, because hospitals, neighborhoods, and ZIP codes have different baseline risks. The result is a mixture of Poissons, which is mathematically a negative binomial, and the variance is meaningfully bigger than the mean. The naive Poisson test should fail. I want to see how loudly.

```python
# fit Poisson to a count table, return chi-square stats
ks = np.array(sorted(counts.keys()))
obs = np.array([counts[k] for k in ks], dtype=float)
n = obs.sum()
lam = (ks * obs).sum() / n
exp = poisson.pmf(ks, lam) * n
exp[-1] += n - exp.sum()    # bundle the right tail
chi2 = ((obs - exp) ** 2 / exp).sum()
```

About sixty lines of Python including the bin-merging logic that chi-square needs when expected counts get small.

## What happened

The Bortkiewicz fit reproduced cleanly. Chi-square = 1.95 on 2 degrees of freedom, p = 0.377. Variance-to-mean ratio = 0.76 / 0.70 = 1.09. The expected and observed rows line up at the level of integers.

![Poisson fits to four count datasets: Bortkiewicz horse-kicks, synthetic Geiger decay, homogeneous rare disease, and heterogeneous rare disease.](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/40_poisson_fits.png)

| Dataset | n | λ̂ | Var/Mean | χ² | dof | p |
|---|---:|---:|---:|---:|---:|---:|
| Bortkiewicz 1898 | 280 | 0.70 | 1.09 | 1.95 | 2 | 0.38 |
| Synthetic Geiger | 2608 | 3.84 | 1.02 | 8.35 | 10 | 0.59 |
| Disease, homogeneous | 500 | 1.21 | 0.91 | 5.95 | 3 | 0.11 |
| Disease, heterogeneous | 500 | 1.31 | **1.82** | 124.0 | 4 | < 1e-25 |

Two things to point at.

The horse-kick deaths really are Poisson. The reason this works is mechanistic. Each Prussian cavalry corps had a few thousand soldiers and horses. Any given soldier-horse interaction had a vanishingly small chance of ending in a fatal kick. Across a corps-year, the count is a sum of many tiny independent Bernoulli trials. That is exactly the regime Poisson's 1837 limit theorem covers. The corps were also roughly comparable in size and training, so the underlying λ was approximately constant across the panel. Both conditions for the law to apply held in the Prussian army for twenty years.

The heterogeneous disease panel failed loudly, which it had to. With variance 2.38 against a mean of 1.31, the variance-to-mean ratio came in at 1.82. The chi-square clocked 124 on 4 degrees of freedom. The observed row has 187 region-years with zero cases against an expected 135, and 14 region-years with five or more cases against an expected 5. That excess in both tails (more zeros, more high counts) is the fingerprint of a Poisson mixture. Real disease counts look like this. Insurance claims look like this. Highway accident counts by intersection look like this. They are not Poisson, they are negative binomial, and the variance-to-mean ratio is the simplest diagnostic.

The synthetic Geiger panel and the homogeneous disease panel both passed at the levels you would expect from sampling noise. The Geiger fit landed at p = 0.59 with 10 cells of frequency data to chew on. The homogeneous panel landed at p = 0.11, with the slight under-dispersion (Var/Mean = 0.91) sitting within the binomial standard error for n = 500.

What I found most striking on rerunning the horse-kick numbers is how brittle the conditions are. If you change one thing about the Prussian army (say, one corps has a sudden epidemic of unbroken remounts), you immediately get over-dispersion and the Poisson fit drops out. The fact that Bortkiewicz's data passes is partly a statement about probability and partly a statement about the bureaucratic uniformity of the late-nineteenth-century Prussian military.

## What it may mean

The law of rare events works exactly where its assumptions hold, and the assumptions are sharp. You need many trials, you need a small per-trial probability, and crucially you need that probability to be the same across the panel. If any one of those fails, Poisson breaks.

The practical version of this, for anyone with count data, is the variance-to-mean ratio. Compute it. If it is close to 1, Poisson is a reasonable starting model. If it is meaningfully above 1, switch to negative binomial. The disease panel here gave Var/Mean = 1.82, and the naive chi-square test detected the deviation at p essentially zero with n = 500. That is the kind of red flag that should change which model you publish.

For epidemiology and hospital-admissions modeling, this matters because confidence intervals computed under a Poisson assumption are too narrow when the data is over-dispersed. You get false significance for things that should not be significant. The 2003 SARS literature, the 2020 COVID early-spread literature, and most of the highway-safety literature carry this issue in the background.

## Loose ends

I did not pull the original 1898 Bortkiewicz tables; I used the standard textbook 144/91/32/11/2 row, which has been quoted enough times that the numbers may be slightly polished. With another week I would scan the German original from *Das Gesetz der kleinen Zahlen* and re-extract the corps-by-corps tallies, then run the fit corps by corps to check whether some Prussian cavalry units were more dangerous than others. I would also run a formal likelihood-ratio test against negative binomial on the disease panel rather than reading the variance-to-mean ratio off the table. The whole pipeline runs in under two seconds on a laptop, so anyone curious can replicate the horse-kick fit in roughly five minutes.
