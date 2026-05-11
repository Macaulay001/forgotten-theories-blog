# I simulated Hick's 1952 reaction-time law. The slope came back at 152 ms per bit.

*Part 48 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1952, William Edmund Hick sat people in front of a row of lamps and
asked them to press a key under whichever one lit up. He varied the
number of lamps and timed the responses. The next year, Ray Hyman ran
a tighter version of the same experiment with unequal lamp probabilities.
Both reported the same odd, clean result: mean reaction time grew as a
straight line in the *information content* of the choice. Specifically:

    RT = a + b * log2(N + 1)

for N equiprobable alternatives. The slope b ran around 150 ms per bit
of decision. It was one of the first times Shannon's brand-new
information theory (1948) had been pinned to a behavioral measurement,
and it became a load-bearing equation in human factors engineering.
Then it mostly slid out of the cognitive science spotlight, partly
because some practiced subjects could flatten the slope to nearly zero,
and partly because nobody wants to keep citing a 1952 lamp study.

I wanted to know if the original numbers would fall out cleanly when you
simulate the experiment under the generative assumption Hick wrote down,
with realistic noise. And, separately, whether Hyman's entropy
generalization actually buys you anything when probabilities are skewed.

## What I tried

The plan was small. Set ground-truth parameters in the range Hyman's
subjects produced: intercept a = 190 ms (non-decision time, mostly
perception plus motor execution), slope b = 155 ms per bit, per-trial
Gaussian noise of 40 ms on top of the mean. Pick N values from 1, 2, 4,
8, 16. Simulate 80 trials per N. Fit a linear regression of RT against
log2(N+1) and see how close the recovered (a, b) come to the truth,
and what fraction of variance the line explains.

Then a second phase. Build seven mixture distributions where one option
hogs most of the probability mass. For each, compute the Shannon entropy
H = -sum p log2(p). Simulate RTs from a generative model that is linear
in H (this is Hyman's claim). Then fit two competing lines: RT vs H, and
RT vs log2(N), where N is just the number of options with nonzero
probability. If Hyman was right, H should win. If Hick's original
log2(N) form had captured the essential variable, the two fits should be
comparable.

The whole thing fit in about 100 lines of numpy. No special libraries,
no LLM in the loop for this one, just a random number generator and
least squares.

```python
A_TRUE, B_TRUE, SIGMA = 0.190, 0.155, 0.040
Ns = np.array([1, 2, 4, 8, 16])
for N in Ns:
    mu = A_TRUE + B_TRUE * np.log2(N + 1)
    rts = RNG.normal(mu, SIGMA, size=80)
    ...
# Fit RT = a + b * log2(N+1)
A = np.vstack([bits, np.ones_like(bits)]).T
(b_fit, a_fit), *_ = np.linalg.lstsq(A, all_rt, rcond=None)
```

That is essentially the whole experiment. Everything below is what came
out of running it.

## What happened

Phase 1 recovered Hick almost exactly.

| quantity      | true     | fit recovered |
|---------------|----------|---------------|
| intercept a   | 190 ms   | 196 ms        |
| slope b       | 155 ms/bit | 152 ms/bit  |
| R^2           |    -     | 0.949         |

The fit pulled the slope within 2% of the seeded truth and explained
about 95% of variance across roughly 400 simulated trials. That number
matches the kind of fit Hyman reported on his cleaner subjects (he had
some R^2 values above 0.99, but his per-subject noise was lower than
the 40 ms I seeded here). A slope of 152 ms/bit lands inside the
110-220 ms/bit range Hyman tabulated.

![Hick law fit](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/48_hick_fit.png)

The left panel of the figure shows the five condition means as black
points with standard-error bars, plus the fitted line. The visual story
is the boring kind of pleasing: the points sit on the line, the line
goes through the origin region near the intercept, the slope is what
the textbook said it would be.

Phase 2 is where the more interesting answer lives. With unequal
probabilities, log2(N) is the wrong axis and entropy H is the right one.

| predictor on RT  | R^2    |
|------------------|--------|
| log2(N)          | 0.194  |
| Shannon H        | 0.912  |

When one alternative carries 85% of the probability in an N=8 task, H
drops to about 1.0 bit. Subjects (in the simulation, but also in real
data per Hyman) respond as fast as in a near-deterministic two-choice
trial, because they have an internal prior on the high-probability
option. The log2(N) form keeps insisting the task carries 3 bits because
there are 8 lamps, and its predictions miss by 100+ ms. Hyman's entropy
extension is not a minor refinement; it is the difference between a
model that works and a model that doesn't, once you leave the
equiprobable case.

What strikes me about this re-running is how *cheap* it was to confirm.
The 1952 paper required hours of bench experimentation per subject,
plus careful equipment to time button presses to the millisecond. The
generative model fits in three lines. The fit reproduces seven decades
of psychophysics in seconds. That asymmetry is part of why some old
laws get forgotten: they were hard-won once, then absorbed so completely
into the textbooks that the original measurement falls out of memory.

## What it may mean

If the simulated picture matches reality (and the bulk of evidence
suggests it roughly does for unpracticed subjects on choice tasks),
then RT-vs-information is one of the cleanest quantitative laws in
behavioral science. The slope b is roughly invariant across modality,
within a subject, on standard lab tasks. It appears to shift with
practice, age, and certain neurological conditions. A few labs have
proposed using Hick slope as a cheap, equipment-light cognitive
biomarker for early Parkinson's or for tracking medication effects in
schizophrenia.

I am not sure how well that holds up outside the lab. The
intra-subject variance from session to session can easily swamp the
between-condition effect for a single individual. A clinical-grade RT
test would need either many trials per session or pooled multi-session
estimates of the slope. None of that is impossible. None of it is
free either, and the cost of a careful longitudinal study with even a
few dozen patients runs into real money. Still, the equipment side is
trivial: a laptop, a keyboard, a stopwatch routine, and the same
linear regression I just ran.

What seems clearer is that Shannon information, defined for telephone
wires in 1948, predicts how long a human takes to pick between lamps in
1952. That coincidence does not get less strange the longer I look at
it.

## Loose ends

This is a simulation, not a re-run of Hick's actual data. The numbers
recover because the generative model was set up to recover them. The
honest extension would be to pull a public choice-RT dataset (the
Cambridge Brain Sciences group has released some, and the Hyman 1953
tables are still in PDF form) and fit the same two lines. With another
week I would do that, plus test the practiced-subject regime where the
slope is reported to approach zero. I would also include stimulus-
response compatibility (Fitts and Seeger 1953), which changes the
intercept without touching the slope, and ask whether that decomposition
survives modern analysis. If you have a clean RT dataset with varying
N, please get in touch.
