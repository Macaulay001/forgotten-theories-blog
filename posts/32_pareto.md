# I checked Pareto's 80/20 rule on five distributions. The actual numbers landed between 80 and 91.

*Part 32 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1896, the Lausanne economist Vilfredo Pareto was working through Italian tax records and noticed something that bothered him. Land ownership was not just unequal in the obvious sense. It was unequal in a way that repeated itself. About 80% of the land sat in the hands of roughly 20% of the families, and when he checked Prussia, England, and a scattering of other places, similar splits kept showing up. He generalized this into a curve, what we now call the Pareto distribution, and the working version became "the 80/20 rule".

Joseph Juran, the quality-control engineer, picked it up in the 1940s and recast it as the "vital few and the trivial many". From there it spread. 80% of bugs from 20% of code. 80% of sales from 20% of customers. 80% of crime from 20% of offenders. It is now folk knowledge. It is also rarely tested with numbers that anyone bothers to publish.

I had a slow afternoon, so I tested it.

## What I tried

The plan was simple. Pull five distributions that should, if Pareto was on to something general, all show roughly 80/20 splits. Sort each one from largest to smallest. Sum the top 20% of units. Divide by the total. See where the percentages actually land.

The five datasets:

1. **File sizes** on my local theories tree. A `walk` of the directory, grabbing every file size in bytes. 12,814 files in the end, totaling about 5.6 GB.
2. **GitHub contributor commits**, synthesized. I drew 500 commit counts from a Zipf process tuned so the resulting Gini coefficient roughly matched what published surveys report for active open-source projects (around 0.7 to 0.8).
3. **Linux kernel author commits**, also synthesized. 4,000 authors, heavier tail, calibrated so the top handful of names account for the majority of commits, which matches Jonathan Corbet's annual kernel author analyses.
4. **Word frequencies** in Moby Dick. Real data, reused from an earlier piece on Zipf's law. 17,140 distinct word forms.
5. **Wealth or income**, synthesized from a lognormal with sigma 1.7. That gives a Gini around 0.76, which is in the neighborhood of US household wealth (depending on the source).

I would have preferred a real GitHub contributor pull and a real kernel author dump. I didn't have those today. The synthetic substitutes are calibrated to published Gini values, so the qualitative shape is right even if the second-decimal precision is not.

For each dataset, three numbers: the share of the total captured by the top 20% of units, the Gini coefficient, and the Pareto alpha that would produce the observed split if the underlying distribution were exactly Pareto. The relevant identity is that a Pareto with shape alpha gives a top-fraction-q share equal to q to the power of (alpha-1)/alpha. Solve for alpha and you can compare distributions on a common scale.

## What happened

Here is the working core of the script. Sort, slice the top fifth, sum, divide.

```python
def top_share(values, frac=0.2):
    v = np.sort(np.asarray(values, dtype=float))[::-1]
    k = max(1, int(round(frac * len(v))))
    return v[:k].sum() / v.sum(), k, len(v)

def implied_alpha(share, frac=0.2):
    r = math.log(share) / math.log(frac)
    return 1.0 / (1.0 - r)

for name, vals in datasets.items():
    s, k, n = top_share(vals)
    print(f"{name}: n={n}, top-20% share = {s*100:.1f}%, "
          f"alpha = {implied_alpha(s):.2f}")
```

The results, in one table:

| Dataset | n | top-20% share | Gini | implied alpha |
|---|---:|---:|---:|---:|
| Local file sizes | 12,814 | 86.6% | 0.845 | 1.10 |
| GitHub contributors (synth) | 500 | 84.5% | 0.786 | 1.12 |
| Kernel commits (synth) | 4,000 | 91.4% | 0.887 | 1.06 |
| Word frequencies (Moby Dick) | 17,140 | 88.1% | 0.850 | 1.09 |
| Wealth (lognormal synth) | 10,000 | 79.7% | 0.763 | 1.16 |

Five datasets, top-20% shares between 79.7 and 91.4. Mean of 86.1%. The classic "80" is on the low end of what these distributions actually do.

A clean Pareto distribution gives exactly 80/20 when alpha equals log(5)/log(4), which is about 1.161. None of these distributions hit alpha = 1.161 on the nose. They mostly cluster between 1.06 and 1.16, slightly more concentrated than the folk number predicts.

The Lorenz curves tell the same story visually. Plot the cumulative share of units on x and the cumulative share of the total on y, sorted from smallest unit upward, and you get a curve that sags away from the diagonal of perfect equality. The deeper the sag, the more concentrated the distribution. All five curves sit well below the diagonal. The one that comes closest to the 80/20 reference is the synthetic wealth lognormal, which clocks in at 79.7% (Gini 0.76). The most extreme is the synthetic kernel commit set at 91.4%, where the top 20% of authors do almost everything and the bottom 80% are mostly drive-by contributors.

The interesting thing is the spread. People quote 80/20 as if the second number is a constant of nature. Across the five datasets I looked at, the value of "the 80" varies by about 12 percentage points. That is a lot if you are budgeting effort. Treating a true 90/20 distribution like an 80/20 one means you leave 10% of the action uncovered when you focus on the top fifth. In a fraud detection setting that 10% is real money.

But the qualitative rule, that 20% of units carry the bulk of the load, survives. None of these distributions are 50/20 or 60/20. The "vital few" framing seems to hold even when the exact share drifts.

One more sanity check. The word frequency data is the only one from a non-synthetic source, and it ends up at 88.1%, close to the synthetic distributions. That gives me a little confidence that the synthetic generators are not where the result is coming from. A real heavy tail seems to land in roughly the same place as a tuned Zipf draw.

## What it may mean

If you treat 80/20 as a precise prediction, it fails on most of these distributions. If you treat it as a centering principle, it holds. The folk version of Pareto's rule appears to be a slightly conservative version of what real heavy-tailed processes do. Three of the five datasets I checked are *more* concentrated than 80/20, not less. So the people quoting it in business books are, if anything, underselling the inequality of their underlying systems.

I cannot rule out that this is partly an artifact of which distributions I picked. All five have heavy tails by design. If I had thrown in human heights or body temperatures, the top 20% would carry close to 20% (which is the equality result). Pareto's rule is not a law about all distributions, it is a regularity about a specific subset, namely the ones that follow power-law or near-power-law shapes. Wealth, file sizes, word frequencies, software contributions, sales per customer all sit in that subset. So do earthquake magnitudes, paper citations, and city sizes, although I did not test those today.

The historical claim, that Pareto noticed something general while looking at one country's land records, holds on this small sample. The number he chose to attach to it (80) is approximately right but treated with more precision than it deserves.

## Loose ends

What I would do with another week: pull real GitHub commit counts from one of the public archives, get the actual Linux kernel author list from the git log (a one-liner), and add a few distributions that *shouldn't* follow Pareto (heights, IQ, blood pressure) as negative controls. I would also bootstrap confidence intervals on each top-20 share, because the differences between 84.5 and 86.6 may not be real at n = 500. None of those changes would, I think, move the basic conclusion: 80/20 is a rough but useful pointer, and the real number is usually a bit higher than 80.
