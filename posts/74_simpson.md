# Berkeley's 1973 admit rates flip sign when you stop pooling the departments

*Part 74 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1903 Udny Yule wrote a short paper on the association between two
attributes in a 2x2 table. He noticed that the sign of the association
could change once the data were broken down by a third attribute. The
result sat in the statistical literature without a name for almost half
a century. Edward Simpson formalised it in 1951, and the paradox now
carries his name (with Yule's contribution sometimes acknowledged in
the hyphenated version).

The textbook one-liner: a trend that appears in each subgroup can reverse
when the subgroups are combined. People hear this and assume there must
be a sleight of hand somewhere. There is not. The arithmetic is fine.
The aggregate just answers a slightly different question than the
per-group rates, and if you forget which question you asked, you can
walk straight off a cliff.

I wanted to reproduce the two canonical cases by hand, then put a
number on how much of the apparent reversal is driven by the confounder
in each case.

## What I tried

Two datasets, both small enough to type into a Python file.

The first is from Bickel, Hammel and O'Connell's 1975 *Science* paper on
graduate admissions at UC Berkeley in fall 1973. The university had
been worried about a possible discrimination suit. The overall admit
rate was 44% for men and 35% for women, a gap large enough to look
serious. Bickel and colleagues went department by department. In most
departments women were admitted at the same rate as men or a little
higher. The aggregate gap was almost entirely produced by women applying
in larger numbers to departments that admitted fewer applicants of
either sex. I used their published Table 1 for the six largest
departments.

The second dataset is the Justice/Jeter batting averages from 1995 and
1996. David Justice outhit Derek Jeter in each year separately. Across
the two years pooled, Jeter outhit Justice. The driver was at-bat
count: Jeter had 582 plate appearances in 1996 (his rookie-of-the-year
season) against Justice's 140. The pooled average weights each player by
their own season totals, so Jeter's combined number sits near his good
year and Justice's sits near his merely-okay one.

For Berkeley I also wanted a clean decomposition of the gap. The natural
counterfactual is: what would women's aggregate admit rate be if they
had the same departmental application mix as men? That isolates a
"rate effect" (would women still be admitted less, at the same dept mix)
from a "composition effect" (do women apply to the harder departments).
I used an Oaxaca-style split at the mean of the two application shares,
which is symmetric and avoids the choice of which group's mix to use as
the base.

## What happened

Both reversals reproduce immediately. The Berkeley aggregate gap is
**+14.2 percentage points** in favour of men. Department by department,
women are at parity or ahead in four of six departments. Department A
admitted **82.4%** of female applicants against **62.1%** of male, a
twenty-point reversal of the headline number on this one department.

![Berkeley 1973 admit rates by department and Justice/Jeter batting averages across 1995-1996](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/74_simpson_panels.png)

The decomposition pins down the mechanism. Splitting the +14.2 pp
aggregate gap at mean application shares, the rate effect comes out at
**-3.6 pp** (women slightly ahead, on a same-mix counterfactual) and the
composition effect at **+17.8 pp**. Composition is larger than the gap
itself; the rate effect is pulling in the opposite direction. Take away
the dept-mix differences and the headline gap turns negative.

Concretely: men sent 1198 admits out of 2691 applications in these six
departments. Women sent 557 out of 1835. The 2691 vs 1835 split is
already roughly 60-40 by gender, but the within-department applicant
splits are wildly different. Department A's applicants were 88% male.
Department E's applicants were 67% female. A and B between them admitted
over 60% of applicants. E and F admitted under 30%. Women preferentially
applied to the harder rooms.

```python
# core of the decomposition: rate vs composition at mean shares
share_men   = m_app / m_app.sum()
share_women = w_app / w_app.sum()
mean_share  = 0.5 * (share_men + share_women)
mean_rate   = 0.5 * (m_rate + w_rate)
rate_effect = (mean_share * (m_rate - w_rate)).sum()
comp_effect = ((share_men - share_women) * mean_rate).sum()
print(rate_effect, comp_effect, rate_effect + comp_effect)
# -0.036  +0.178  +0.142  (matches the aggregate gap)
```

The baseball case is even starker because the at-bat asymmetries are
extreme. Justice hit .253 in 1995 and .321 in 1996. Jeter hit .250 in
1995 and .314 in 1996. Justice was higher in both years. Combined,
Justice hit .270 and Jeter hit .310. Forty points the other way.

| season   | Justice | Jeter |
|----------|---------|-------|
| 1995     | .253    | .250  |
| 1996     | .321    | .314  |
| combined | .270    | .310  |

There is no statistical trick here. A combined batting average is
literally `total_hits / total_at_bats`, which is a weighted mean of the
two season averages where the weights are the at-bat counts. Justice's
combined number is weighted toward his weaker 1995. Jeter's is weighted
toward his stronger 1996. The pooled comparison answers "who had more
hits per at-bat across the two years", and the answer to that question
is genuinely Jeter. It is the per-season comparison that answers "who
hit better when both were swinging", and that answer is Justice. Two
different questions, two different answers. Neither lies.

The Berkeley case has the extra moving part of admission decisions being
made by departments, not by the university as a whole. If you wanted to
ask "is the *university* biased against women", the relevant denominator
is the university's admissions process taken end to end. But the
university does not run that process. Departments do. So the aggregate
rate is the right number for a question nobody is making decisions
about, and the wrong number for any question anyone actually cares
about. Bickel et al. made this point thirty pages into their paper and
it is still mostly missed by writers reaching for the example.

## What it may mean

The paradox does not need a resolution. It is a property of weighted
averages, which is to say it is a property of arithmetic. What it does
do is force a question: what is the right level of pooling for whatever
decision you are about to make.

If the decision is made at the department level (admit this applicant or
not), the department-level rates are the relevant inputs. The aggregate
is a summary statistic that has no decision-maker attached to it. If
the decision is made at the player level (which one do I want on my
roster across the next two years), the pooled average is closer to
what matters than the season-by-season split, because what you care
about is total run production. Different question, different number.

The wider point may be that any time you see a single ratio used to
describe a population, the ratio is implicitly answering a counterfactual
about how the units were sampled. Change the sampling and the ratio
shifts, even when nothing about the underlying mechanism has changed.
Causal-inference frameworks (DAGs, stratification, do-calculus) exist
to make that counterfactual explicit instead of leaving it buried in
whichever denominator was easiest to compute.

In this sample the rate effect at Berkeley actually favoured women by
about 3.6 percentage points once department mix was held constant. That
number is not a finding about Berkeley in 2026, and the original authors
were careful not to read it as evidence of reverse bias. It is a
finding about what the 1973 numbers say once you ask a sharper question
of them.

## Loose ends

The Oaxaca-style decomposition is not unique. Using men's application
mix as the base attributes the entire gap to composition. Using women's
mix attributes most of it to rate. The mean-share split is one common
compromise but the index-number problem is genuine and any single
attribution number should be treated as a convention, not a fact.

The Berkeley reconstruction uses only the six largest departments, the
ones Bickel et al. tabulated. The full 101-department picture has the
same direction with a slightly smaller gap, and a few small departments
do show within-department gaps that would be worth a closer look.
None of that changes the headline reversal. The next thing I would
do with another week is pull the Berkeley admit data from the IPEDS
archive and rebuild the decomposition at the full-department resolution,
which would take roughly an afternoon if the historical files survive.
