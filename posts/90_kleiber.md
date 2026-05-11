# Kleiber's 1932 mass-metabolism slope holds at 0.77 on his own 13 species

*Part 90 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1932, a Swiss-trained agricultural biologist named Max Kleiber
published a paper titled "Body size and metabolism" in *Hilgardia*, a
journal most biologists today have never opened. He had measured how
much heat thirteen animals gave off per day, lined them up by body
weight, and noticed that the relationship was straight on log-log
paper. The slope he reported was about 0.74. That number became known
as Kleiber's law: metabolic rate B scales with mass M as B ∝ M^(3/4).

It is a strange exponent. The old "surface rule" by Max Rubner
(1883) predicted 2/3, because heat loss should scale with surface area
and surface area scales as M^(2/3). Three-quarters has no comparably
simple geometric story. For seventy years it sat in textbooks anyway,
mostly because the data kept coming back near 0.75.

Then in 2003, Craig White and Roger Seymour reanalyzed a larger mammal
dataset in *PNAS*, corrected for body temperature, and reported a
slope nearer 0.67. The headline was that Kleiber was wrong and Rubner
was right. The argument has rumbled along since, with West, Brown and
Enquist defending 3/4 from a fractal-network model and skeptics like
Hulbert (2014) calling the whole thing an artifact of how you fit a
line through scattered points.

I wanted to see what Kleiber's *original* thirteen numbers say when
you fit them with a modern OLS in log space. Not the reanalyses, not
the temperature corrections. His own 1932 table.

## What I tried

I hand-encoded Kleiber's Table 1 from the 1932 *Hilgardia* paper:
thirteen rows of body weight in kilograms and daily heat production
in kilocalories. The species list is eclectic: a dove (0.15 kg), a
rat (0.26 kg), a hen, two dogs of different sizes, a goose, a sheep,
a woman, a man, two cows, a steer, and a heifer. The largest entry
is a 679 kg steer. The smallest is the dove.

For comparison I also built a broader modern sample of 24 mammals
spanning a 0.0035 kg shrew to a 100,000 kg blue whale, with BMR
values taken from published summaries in White & Seymour (2003) and
Savage et al. (2004). The point of the modern set was to ask whether
the slope changes when you extend the mass range by five orders of
magnitude beyond what Kleiber had. I treated each paper-reported BMR
as a point estimate with no within-species variance, which is a
shortcut and worth flagging.

The fit itself is the simplest thing you can do: take log10 of mass
and log10 of metabolic rate, run an OLS line, read off the slope and
its standard error. Then ask whether the 95% confidence interval
contains 0.75, contains 0.667, both, or neither.

## What happened

Kleiber's own thirteen points yield:

```python
logM = np.log10(M)
logB = np.log10(B)
slope, intercept = np.polyfit(logM, logB, 1)
# slope = 0.767
# SE    = 0.012
# 95% CI = (0.741, 0.792)
# R^2   = 0.9975
# t vs 0.75:  +1.43   (inside CI)
# t vs 0.667: +8.64   (way outside CI)
```

So on his own data, the slope sits at 0.767. The 95% confidence
interval comfortably contains 0.75. It excludes 0.667 by more than
eight standard errors. The R^2 is 0.9975, which is almost embarrassing
for a thirteen-point fit across species ranging four orders of
magnitude in mass. Whatever else is going on, the mass dependence is
not noisy.

The broader 24-species modern fit gives a slope of 0.783 with a CI of
0.774–0.791. That CI is *just above* 0.75. R^2 is 0.9993. Both fits
land closer to 3/4 than to 2/3, but the modern sample drifts a little
high, in the range that Savage et al. (2004) reported (0.74–0.78)
for mammals.

| Sample | n | slope | 95% CI | R^2 |
|---|---|---|---|---|
| Kleiber 1932 | 13 | 0.767 | 0.741–0.792 | 0.998 |
| Modern mammals | 24 | 0.783 | 0.774–0.791 | 0.999 |

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/90_kleiber_loglog.png)

The figure shows both samples on a log-log plot with two reference
slopes (3/4 in green, 2/3 in blue) anchored at a 70 kg human.
Kleiber's points hug the 3/4 line for most of the range. The 2/3
reference line drifts visibly below the data at large masses, which
is where the two predictions diverge most.

A few caveats on these numbers. Kleiber's thirteen rows include
within-species pairs (a small dog and a large dog, two cows, a steer
and a heifer), which means the effective sample size is smaller than
13 if you think of mammalian-species variance as the relevant noise.
The CI I report assumes independence across rows. The modern sample
sidesteps that problem but introduces its own: I am quoting
paper-reported means with no within-species variance, and I have not
applied phylogenetic correction. White and Seymour's 0.67 result
required a phylogenetic GLS and a body-temperature adjustment, and on
this analysis neither was attempted.

There is also the choice of regression. OLS in log space minimizes
vertical residuals; reduced-major-axis (RMA) minimizes a symmetric
distance and gives a slightly steeper slope. RMA would push Kleiber's
0.767 to roughly 0.768 here (the R^2 is so high it barely matters),
but on noisier datasets the choice has been part of the 2/3 vs 3/4
debate for decades.

## What it may mean

Taken at face value, Kleiber's 1932 number was not just lucky. His
thirteen-species table, fit with a method his contemporaries would
not have had on hand, gives a slope of 0.77 with a tight confidence
interval that brackets 3/4. A broader modern sample gives 0.78. The
2/3 "surface rule" is outside both intervals on the simplest analysis.

That does not settle the modern argument. The case for 2/3 rests on
two corrections that this analysis ignored: a temperature adjustment
(because BMR depends on the temperature differential between the
animal and its surroundings, and that differential varies with size),
and a phylogenetic correction (because closely related species share
metabolic strategies and shouldn't count as independent points).
Both corrections can flatten the slope. Whether they should is a
biological question, not a statistical one.

If 3/4 is the right answer, it is probably mechanistic. West, Brown
and Enquist's 1997 model derives 3/4 from the geometry of fractal
supply networks (vascular trees, tracheal systems). The model
predicts other quarter-power exponents too: lifespan scales as
M^(1/4), heart rate as M^(-1/4), and so on. Veterinary and pediatric
drug dosing already use M^0.75 allometric scaling, which is one place
where the exponent is not an academic argument. If the exponent
really is closer to 2/3 for some clades, dosing at the extremes
(neonates, megafauna) might need rethinking.

For now, on this sample, the 1932 paper appears to have aged better
than the 2003 reanalysis suggested.

## Loose ends

The biggest thing I did not do is the temperature correction. White
and Seymour built their 0.67 result on it, and dismissing their
analysis without engaging that step would not be fair. With another
week I would pull the AnAge or PanTHERIA database (roughly 600
mammal species with mass + BMR + body temperature), run both the raw
and temperature-corrected fits, and add a phylogenetic GLS using a
standard mammal supertree. That would land the analysis in the same
methodological space as the modern debate, instead of replaying
Kleiber's own arithmetic on his own thirteen rows.

You can replicate this analysis in about ten minutes: the data table
is in `src/fit_kleiber.py` and the fit is a one-line `np.polyfit`.
