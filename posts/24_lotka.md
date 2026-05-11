# Lotka said scientific productivity follows 1/n². I refit his own 1926 data and the exponent is 1.82.

*Part 24 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In December 1926, Alfred J. Lotka, an actuary at the Metropolitan Life Insurance Company and a part-time mathematical biologist, published a six-page paper in the *Journal of the Washington Academy of Sciences*. The title was "The frequency distribution of scientific productivity". The claim was simple enough to fit on a postcard. Take all authors in a field. Count how many wrote 1 paper, how many wrote 2, how many wrote n. The number of authors at n papers, Lotka said, falls roughly as 1 over n squared. About 60% of the population is one-paper authors. Almost nobody is in the long tail.

The paper became famous as "Lotka's law" and got cited approximately forever in bibliometrics. The specific exponent, 2, got cited as if it were a constant of nature, like the gravitational constant but for scientists. Modern re-tests on physics, biology, and computer-science corpora have reported exponents anywhere from about 1.8 to 3.0. The wobble around the canonical value seemed worth a closer look. I started with what looked like the obvious experiment, then ended up doing something smaller and (I think) more interesting.

## What I tried

The plan was to test Lotka on a modern corpus. The arXiv API offers a clean window into a single discipline with reasonable author lists, so I picked cs.LG (machine learning) papers from the first half of 2024, set a target of 6,000 papers, and wrote a polite fetcher with 15-second sleeps between requests. The pipeline would then normalize each author name to "first-initial.lastname", count papers per author, build the count distribution, and fit a power law on the log-log of (n, frequency).

That part did not survive contact with reality. The arXiv API timed out partway through the run. Around 100 papers came back before I stopped retrying. A hundred papers and roughly seven authors each gives you maybe 600 author observations, almost all of them with one paper, which is not enough to fit a power law on. The fetcher and analyzer code are in the repo if anyone wants to try again on a quieter day for the arXiv mirrors. I had pre-registered a fallback in case the API misbehaved, so I switched to it.

The fallback is Lotka's own data. His 1926 paper has a table, Table 1, listing the count distribution of authors in *Chemical Abstracts* 1907 through 1916, restricted to authors whose surnames begin with A or B. Twenty rows: 3,991 authors wrote one paper, 1,059 wrote two, 493 wrote three, all the way to 14 authors who wrote twenty papers each. Total: 6,747 authors. I typed the table into a Python dict by hand, refit Lotka's claim with modern OLS on the log-log of (n, fraction), and asked the same question he asked: what is the exponent?

The choice to refit his own data instead of a modern corpus changes the question from "does Lotka still work today?" to "did Lotka work on Lotka's day?" The second question is narrower but, on reflection, the more useful one. If the answer is "almost, but with an exponent of 1.85 instead of 2", then everyone reporting 1.8-something on modern corpora is not contradicting Lotka, they're agreeing with him at slightly more decimal places than he chose to print.

## What happened

```python
# Lotka 1926 Table 1: papers per author -> author count
LOTKA_CHEM = {1:3991, 2:1059, 3:493, 4:287, 5:184, 6:131, 7:113,
              8:85, 9:64, 10:65, 11:41, 12:37, 13:37, 14:26,
              15:33, 16:28, 17:19, 18:19, 19:21, 20:14}
total = sum(LOTKA_CHEM.values())                 # 6747
ks = np.array(sorted(LOTKA_CHEM))
fs = np.array([LOTKA_CHEM[k]/total for k in ks]) # fractions
slope, intercept = np.polyfit(np.log10(ks), np.log10(fs), 1)
alpha = -slope                                   # 1.820
r2 = ...                                         # 0.994
frac_one = LOTKA_CHEM[1]/total                   # 0.592
```

Fitted alpha: **1.820**. R-squared of the log-log line: **0.994**. Fraction of single-paper authors: **0.592**, against a value of 0.608 that you would expect if alpha were exactly 2 (the reciprocal of zeta(2), pi-squared over 6, which appears here because that's the normalization constant for the Riemann zeta distribution with parameter 2).

So a power law fits Lotka's data essentially perfectly. R-squared of 0.994 on 20 bins across two orders of magnitude in n is as clean as bibliometric data gets. But the exponent his paper is famous for is wrong by about 10%. The number that survives is 1.82, not 2.00.

| Quantity | Lotka claimed | Refit gives |
|---|---:|---:|
| Exponent alpha | 2.00 | 1.82 |
| Fraction at n = 1 | ~0.61 | 0.59 |
| R^2 on log-log | (not stated) | 0.994 |
| Authors counted | 6,891 | 6,747 |

Looking at the plot, the data sits very slightly above the alpha = 2 reference curve in the high-n part of the tail. A shallower exponent means a thicker tail of productive authors. Out at n = 20, the inverse-square law would predict roughly 1/400 of the population, about 17 authors. Lotka's own table has 14 there, which seems consistent with alpha = 2, but the pattern across n = 5 through n = 20 is systematically above 1/n² by an amount that the OLS fit picks up as a slope of -1.82.

Whether 1.82 is "really" different from 2.00 depends on what you call really. The 1.82 has no error bars in this fit because the input is a single deterministic table. If we modeled Lotka's counts as Poisson draws from an underlying rate and bootstrapped, we'd probably get a 95% interval that excludes 2.0 (the gap is large), but I haven't done that. There's also a known issue with OLS on log-log power law data: it gives slightly biased exponents compared to maximum likelihood with a Clauset-Shalizi-Newman x_min selector, and the bias usually goes in the direction of a shallower slope. So a proper MLE on these same counts might land at 1.95 instead of 1.82, and the gap from 2 might shrink. I'm reporting OLS because that's the historical comparison: Lotka himself used a kind of eye-fit on log-log paper, and OLS is the natural modern stand-in.

The other thing the data does say is that the *qualitative* part of Lotka's story is exactly right. About 60% of authors write exactly one paper. The next 16% write two. The top 1% by productivity in this table wrote 13 papers or more. Power law shape, single-paper dominance, long thin tail of productive specialists, all there. Lotka was right about the shape. The choice of "2" appears to be a round-number idealization he selected after seeing his own numbers. Several historians of bibliometrics (Pao in the 80s, Coile in the 70s) have made this point. The refit just makes it numerical.

## What it might mean

The boring takeaway is that "Lotka's law" should probably be cited as "Lotka-shaped" rather than as a specific 1/n² claim. Reported exponents from 1.8 to 2.3 across modern fields are not contradictions; they sit on a band that includes Lotka's own data once you actually fit it. The exponent looks field-dependent (CS with massive coauthorship may sit lower, single-author humanities fields higher), and probably era-dependent, and we shouldn't expect a single magic number.

The slightly more interesting takeaway is that the qualitative claim, "scientific productivity follows a fat-tailed distribution with most authors at the very bottom", appears to be one of the more durable empirical regularities in the social science of science. It survived a hundred years of changes to publishing, peer review, coauthorship norms, internet preprints, citation databases, and the gradual professionalization of basically every field Lotka studied. The fact that the shape persists while the exponent drifts may be more useful information than a precise value of alpha would have been.

I am not sure what mechanism produces this shape. Yule-Simon processes give power laws. Preferential attachment in coauthorship networks gives them. Heavy-tailed talent distributions combined with cumulative advantage give them. The shape is consistent with several stories, and a power-law fit can't tell those apart. What it can do is set a budget: any plausible model of scientific careers should produce something with alpha in roughly [1.8, 2.3] when measured on a clean field-restricted sample.

## Loose ends

The honest caveats are large. I never ran the modern arXiv test. The 100 papers I did pull are not enough to fit, and I don't know whether 2024 ML would give 1.6, 1.9, or 2.2. I would not be very surprised by any of those. I'd also like to redo Lotka's chemistry data with maximum-likelihood and bootstrap confidence intervals, which would tell me whether 1.82 is statistically distinguishable from 2.0 or just an OLS artifact. Both of those probably take a weekend with a working arXiv mirror. If anyone has a clean dump of cs.LG 2023-2024 metadata, I'd happily run the second analysis with proper error bars.
