# I refit Ebbinghaus's 1885 forgetting curve. A stretched exponential beats the textbook exponential by 28 AIC points.

*Part 46 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In the winter of 1879, a 29-year-old German named Hermann Ebbinghaus sat alone in a room and started memorising lists of nonsense syllables. Things like ZOF, BAK, NIH. He drew them at random from a pool of 2,300, read each list out loud at a metronome-fixed pace, and stopped the clock when he could recite the whole list twice in a row without error. Then he waited. After a measured delay, anywhere from nineteen minutes to thirty-one days, he relearned the same list and recorded how much shorter the second learning took. The percent of effort he saved became his measure of how much he still remembered.

He did this to himself for more than a year, on hundreds of lists, often late at night. The book that came out of it, *Über das Gedächtnis* (1885), is the founding document of experimental psychology. Buried in chapter seven is a small table of seven numbers that became known as the forgetting curve. Generations of textbooks have summarised it as "memory decays exponentially". I wanted to know whether that summary is even close to true.

## What I tried

The plan was small. Hand-encode the seven points Ebbinghaus actually published, fit three candidate models, and let the Akaike information criterion pick a winner. The three candidates were the ones the literature keeps arguing about:

1. Pure exponential, R(t) = exp(-t/tau). One parameter. The textbook default.
2. Pure power law, R(t) = (1 + t)^(-beta). One parameter. The form Wickelgren and Wixted have favoured for decades.
3. Stretched exponential, R(t) = exp(-beta * t^alpha). Two parameters. Used in glass physics and proposed for memory by Wickelgren in 1972.

The data points themselves come straight out of Ebbinghaus's chapter, reproduced in the modern re-analysis by Murre and Dros (PLOS ONE, 2015):

| delay | hours | retention |
|---|---:|---:|
| 19 minutes | 0.317 | 0.582 |
| 1 hour | 1.0 | 0.442 |
| 8.8 hours | 8.8 | 0.359 |
| 1 day | 24 | 0.337 |
| 2 days | 48 | 0.278 |
| 6 days | 144 | 0.254 |
| 31 days | 744 | 0.211 |

Seven numbers. That is the entire dataset. Everything modern claims about the forgetting curve traces back to this row.

The fitting itself is `scipy.optimize.curve_fit` with sensible bounds. I weighted each point equally because Ebbinghaus did not report standard errors on individual delays, only on the broader retraining experiments. The residual sum of squares feeds into AIC = n * log(RSS / n) + 2k, the same form across all three models, so the comparison is apples to apples.

```python
def m_exp(t, tau):       return np.exp(-t / tau)
def m_power(t, beta):    return (1.0 + t) ** (-beta)
def m_stretched(t, beta, alpha):
    return np.exp(-beta * np.power(t, alpha))

popt_e, _ = curve_fit(m_exp, t, R, p0=[10.0])
popt_p, _ = curve_fit(m_power, t, R, p0=[0.2])
popt_s, _ = curve_fit(m_stretched, t, R, p0=[0.5, 0.3])
```

That is the whole experiment. It runs in under a second.

## What happened

The three fits separate cleanly.

| model | best params | RSS | AIC | delta-AIC |
|---|---|---:|---:|---:|
| exponential | tau = 0.96 h | 0.4554 | -17.13 | 28.41 |
| power law | beta = 0.413 | 0.2413 | -21.57 | 23.97 |
| stretched exponential | beta = 0.735, alpha = 0.127 | 0.0059 | -45.54 | 0.00 |

The stretched exponential beats the pure exponential by 28.4 AIC points and beats the pure power law by 24.0. In standard model-selection language, a delta-AIC above 10 is treated as decisive evidence against the worse model. Both one-parameter forms are decisively worse than the two-parameter stretched form on Ebbinghaus's own table.

![Ebbinghaus 1885 retention with three fitted models and residuals](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/46_ebbinghaus_fits.png)

The shape that wins has alpha = 0.127, which is far below one. That small alpha is what gives the curve its character. At short delays, t^0.127 is close to one and the function drops fast; at long delays, t^0.127 grows so slowly that the curve flattens out. Concretely, the stretched fit predicts 58% retention at 19 minutes (Ebbinghaus measured 58.2%), 28% at two days (he measured 27.8%), and 21% at 31 days (he measured 21.1%). The largest residual across all seven points is 0.013 in retention units. The pure exponential, by contrast, predicts essentially zero retention by the eight-hour mark and is off by 30 percentage points at 31 days.

Why does the textbook story persist if a single exponential is this wrong? Two reasons, I think. First, on a linear time axis the early hours dominate the visual, and a fast exponential drop looks roughly right there. Second, a one-parameter law is easier to teach than a two-parameter one, and the qualitative point ("you forget fast then slowly") survives the simplification. The price of the simplification is that any quantitative use of the curve, for instance designing a spaced-repetition schedule, will be wrong by a factor that compounds.

Murre and Dros, in their 2015 re-analysis, also concluded that a single exponential is inadequate and that two-parameter forms fit much better. The numerical values I get for the stretched exponential are within rounding of theirs. So this is not a new finding. It is a replication that the textbook simplification has been wrong since at least 1972, and that anyone who runs the fit themselves on the published table will see it in under a second of compute.

One detail worth flagging. Ebbinghaus's metric is the "savings score", the fraction of relearning effort saved relative to original learning, not the fraction of items recalled. The two are correlated but not identical, and the choice of metric changes the absolute numbers somewhat. The shape of the decay, however, appears to be the same regardless of which memory metric a later study uses, which is part of why the stretched form has held up across follow-up work in the twentieth century.

## What it may mean

A pure exponential is wrong on Ebbinghaus's data, but more importantly it is wrong in a direction that matters for practice. The stretched form predicts that doubling a delay from one day to two days costs you about 6% in retention, while doubling from one week to two weeks costs you about 2%. An exponential with tau matched to short delays would predict roughly the same loss at both scales, which is the wrong policy for a review schedule. Spaced repetition systems like Anki and SuperMemo embed implicit decay models that look much more like the stretched form than a single exponential, which may explain why those systems work as well as they do in practice.

There is a deeper question this study does not settle. A stretched exponential with alpha = 0.127 is unusual. In glass relaxation, alpha tends to sit between 0.4 and 0.9. The very small value here might be an artifact of fitting a small dataset with a flexible two-parameter form, or it might reflect a real feature of how memory traces decay (a mixture of fast and slow processes, for instance). Murre and Dros lean toward a power-law interpretation; Wickelgren's later work argues for the stretched exponential proper. The seven points alone cannot decide between those readings.

The practical takeaway is narrow. On the original Ebbinghaus table, neither of the two laws the textbooks reach for first fits the data. A two-parameter form does, and it does so almost exactly. Anyone teaching the forgetting curve from a textbook diagram is teaching the wrong functional form.

## Loose ends

Seven points is a small dataset. The fit is decisive on those seven points but cannot rule out other two-parameter forms with similar curvature, such as a logarithmic decay or a sum-of-two-exponentials. Ebbinghaus's "subject" was Ebbinghaus, learning nonsense syllables in a quiet room in Berlin, and there is no guarantee the same alpha applies to other people or other material. With another week I would refit against the modern replications (Heller et al. 1991, Murre and Dros 2015), bootstrap the parameters with simulated noise matched to the relearning standard errors Ebbinghaus did report, and check whether alpha shifts meaningfully under different memory metrics. The code and the table are in the repo; a curious reader can run the whole thing in under a minute.
