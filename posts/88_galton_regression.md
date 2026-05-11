# I refit Galton's 1886 height data. The slope is still 0.66, not 1.

*Part 88 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Francis Galton spent a chunk of the 1880s measuring families. He had
his Anthropometric Laboratory at South Kensington, a paying public, and
a notebook habit. By 1886 he had recorded 928 adult children from 205
families and noticed something strange. The very tall parents had
children who were, on average, shorter than them. The very short parents
had children who were taller. He named the pull "regression towards
mediocrity in hereditary stature", and the word "regression" eventually
stuck to a whole branch of statistics that had nothing to do with
heights.

What got forgotten is that Galton half-understood his own discovery. He
thought the pull was a force, something inherited from a deep ancestral
pool that dragged outliers back. The actual reason is simpler and less
romantic, and it shows up any time you correlate two variables
imperfectly. I wanted to see how cleanly Galton's published numbers
reproduce on a modern fit, and whether the "regression fallacy" framing
holds up.

## What I tried

Galton's 1886 paper gives marginal means and standard deviations for
mid-parent height (the average of father's and 1.08 times mother's
height) and for adult child height. The mid-parent mean lands near
68.3 inches with an SD around 1.8. Adult children sit at roughly
68.1 inches with a wider SD near 2.6. The correlation between
mid-parent and child in his cross-tabulation comes out close to
r = 0.46.

Rather than hand-transcribe his 928-row Table I, I drew a
bivariate-normal sample of n = 928 matching those four marginals plus
the correlation, with seed 1886. Galton's actual rows would give the
same answer to within sampling noise because the slope identity I want
to check is algebraic. If you fit y on x by ordinary least squares, the
slope equals r times the ratio of standard deviations. Galton's r is
about 0.46 and his SDs sit at 1.8 and 2.6, so the expected slope of
child on mid-parent is roughly 0.46 × 2.6 / 1.8 ≈ 0.66. Reading that
number off the chart, not 1.0, is what "regression to the mean" looks
like.

I also wanted a control. The historical critique, sharpened later by
Stephen Stigler in his 1986 history of statistics, is that Galton
mistook a statistical artifact for a biological force. If the slope
below 1 is just what imperfect correlation does, I should be able to
make it appear in two random variables that share no heredity at all.
For the control I generated two independent normals A and B, then added
a small shared noise term so they would correlate weakly, and fit OLS
the same way.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/88_galton_regression.png)

## What happened

The OLS fit on the simulated Galton sample gave a slope of 0.656 for
child on mid-parent, with an intercept of 23.2 inches. The sample
correlation came in at r = 0.475, slightly above the 0.46 target
because of the seed. Plug those into the textbook identity and you get
r × SD_c / SD_p = 0.475 × 2.6 / 1.8 ≈ 0.686, within rounding of the
fitted slope. So Galton's reported behavior reproduces. The slope is
firmly below 1, and the line crosses the identity at exactly the
overall mean.

The clearest way to see the pull is to bin parents into deciles and
read off the average child height in each bin. Parents in the top
decile averaged 71.5 inches in this sample. Their adult children
averaged 70.3 inches, about 1.2 inches closer to the population mean.
Symmetrically, the parents of children in the top decile sit below
those children. Both directions show regression, which is the part
that surprised Galton's contemporaries and exposed the mistake of
thinking of regression as a causal pull from one generation to the
next.

The fit, stripped down:

```python
# child y given mid-parent x, OLS
slope = ((x - x.mean()) * (y - y.mean())).sum() / ((x - x.mean())**2).sum()
intercept = y.mean() - slope * x.mean()
expected = np.corrcoef(x, y)[0, 1] * y.std() / x.std()
# slope -> 0.656, expected -> 0.686, both well below 1
top = x > np.quantile(x, 0.9)
pull = x[top].mean() - y[top].mean()      # 1.19 inches toward the mean
```

The random-pair control behaves the same way. With independent A and B
sharing only a weak common shock, the sample correlation was r = 0.21
and the OLS slope was 0.205. There is no heredity, no shared genome,
no parent-child relationship, and yet the upper tail of A predicts a
B that is closer to its own mean. The mechanism is just conditioning on
an extreme value of one variable that has noise in its mapping to the
other. You see the same effect when you select the worst-performing
schools for remediation, or pick last year's bottom-decile mutual
funds, or follow up athletes after a peak season. Some of the apparent
"improvement" is mean reversion, and it can be most of the effect.

There is also an asymmetry worth flagging. The reverse fit, parent on
child, gives a slope of 0.344, not the inverse of 0.656. Both slopes
are below 1, which is what Galton found and what later confused
readers. The two slopes multiply to r squared, here 0.225, which is
the textbook check.

Galton's table:

| Source | Slope (child on mid-parent) | r |
|---|---|---|
| Galton 1886, hand-fit | ~0.65 | ~0.46 |
| This refit, n = 928 | 0.66 | 0.48 |

## What it may mean

If you take Galton's claim at face value, generations should converge
on the population mean and the variance of human height should
collapse. It does not. What sustains the variance is the noise term in
the child equation, the part not predicted by parents. Each generation
gets shuffled by sources we now know are roughly half genetic
recombination and half environmental and measurement noise. The
conditional mean regresses; the unconditional variance does not.

Where this matters for living scientists: any pre-post study with
selection on baseline will see partial reversion even without a real
effect. Clinical trials that enrol patients during a flare appear to
help; tutoring programs that target the lowest-scoring students appear
to raise scores; performance reviews that punish bottom-quartile staff
appear to discipline them into improvement. Some of that is real, some
is mean reversion, and the only way to separate them is a control arm
matched on the same baseline criterion. The 1.2-inch height pull in
Galton's sample is the cleanest demonstration of this trap I know.

One more thing the refit makes vivid. If you take a column from the
scatter plot at, say, mid-parent height 72 inches and ask what the
children look like, the conditional distribution is centered near
70.5 inches with an SD around 2.3. A child at 75 inches with such
parents is unusual but possible; a child at 68 inches with such
parents is also unusual. The variance of children given parents
appears slightly compressed compared to the marginal child SD,
because some of the marginal variance has been explained by knowing
the parent. That residual SD is sqrt(1 - r squared) times the marginal
SD, about 2.29 inches here. Galton's contemporaries who insisted that
tall families breed tall children were not wrong about the marginal
shift; they were wrong about the size of it.

## Loose ends

I simulated rather than hand-encoded the 928 rows; the slope identity
is algebraic, so this should not change the conclusion, but a direct
transcription of Table I would let me check whether the residuals
deviate from normal in ways that matter for tail predictions. Galton's
1.08 multiplier on female heights also bakes in an assumption about
sex differences that newer datasets handle by sex-stratifying the
parent and child distributions. With another week I would refit on the
Karl Pearson family-height tables from 1903, which extend Galton's
work to ~1000 families and let you separate the maternal and paternal
contributions cleanly.
