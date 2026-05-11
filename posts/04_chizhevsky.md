# I gave Chizhevsky's heliobiology a fair test. Something is there, but I'm not sure what.

*Part 4 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1924 a Soviet biophysicist named Aleksandr Chizhevsky published a small book in Kaluga arguing that the 11-year sunspot cycle correlates with the timing of wars, revolutions, and mass unrest on Earth. He claimed that 60 to 80 percent of major historical events fell within three years of a solar maximum. In 1942 Stalin had him arrested. He spent eight years in the Gulag and eight more in internal exile. His work was never properly translated into English until the 1980s, and by then the idea had drifted into the orbit of astrology, biorhythms, and the kind of magazine where pyramid power gets a column. Mainstream social science never picked it back up.

I want to be very clear before going further. What follows is correlational. I did not test a mechanism. The literature around this claim is polluted by people who think the planets steer their love life. I went in expecting to bury the hypothesis.

## What I tried

The plan was modest. Pull the cleanest sunspot series available, pull the most carefully curated conflict datasets, line them up by year, and run the kinds of tests that did not exist in 1924.

For solar activity I used the SIDC/SILSO V2 yearly mean sunspot number, which runs from 1700 to 2025. For conflict, I used two independent sources that historians and political scientists actually argue about: the Correlates of War project (interstate wars 1816 to 2007, plus intrastate wars, around 429 onsets total), and UCDP/PRIO Armed Conflict Dataset v24.1 (1946 to 2023, 299 unique conflicts). Onset is defined as the earliest start year of each conflict. I deliberately avoided any list curated for the purpose of this test. CoW was assembled by political scientists who, as far as I know, have never heard of Chizhevsky.

The overlap window for the heaviest tests is 1818 to 2007, around 190 years. That covers roughly seventeen complete solar cycles.

I built an annual table with four columns: year, sunspot number, CoW onset count, UCDP onset count. Then I ran five tests:

- Pearson r between sunspot number and onset count, raw and after subtracting a 31-year running mean to kill the long-term modernity trend.
- Cross-correlation at lags from minus ten to plus ten years.
- Superposed-epoch analysis. Find the solar maxima (local peaks with SN above 50, found 18 of them between 1830 and 2014), stack the years from minus five to plus five around each peak, and average the onset counts at each lag.
- A circular Rayleigh test on cycle phase. For each conflict, compute its phase angle within the enclosing peak-to-peak interval, and test whether the resulting distribution is uniform.
- Granger causality at lag 2.

![Figure 3. Cycle-phase distribution of conflict onsets (Rayleigh test p < 0.01).](https://gist.githubusercontent.com/Macaulay001/f59401e6fd96a6d5279ca1a54db1e81c/raw/fig3_phase.png)
*Figure 3. Cycle-phase distribution of conflict onsets (Rayleigh test p < 0.01).*


*Figure 3. Cycle-phase distribution of conflict onsets (Rayleigh test p < 0.01).*


*Figure 3. Cycle-phase distribution of conflict onsets (Rayleigh test p < 0.01).*


The Rayleigh test is the one that matters most for Chizhevsky's actual claim, because his claim is about phase clustering, not about a clean sinusoidal correlation.

## What happened

The simple Pearson r between sunspot and CoW onsets after detrending came out at r = +0.13, which is not significant and would, on its own, kill the hypothesis. I almost stopped there.

The phase test told a different story.

```python
peaks = find_solar_maxima(years, sunspot, prominence=50)
phases = cycle_phase(years, [p[0] for p in peaks])
# weight each year's phase by the number of conflict onsets that year
ph = np.array([phases[i] for i, c in enumerate(cow) for _ in range(int(c)) if not np.isnan(phases[i])])
R = np.abs(np.mean(np.exp(1j * ph)))
p_ray = rayleightest(ph)
mean_phase = np.degrees(np.angle(np.mean(np.exp(1j * ph))))
```

For the CoW onsets, the Rayleigh test came back with p = 0.0001 and a mean phase of 38 degrees, which corresponds to roughly one year after the solar peak. For UCDP it was p = 0.0052 with a mean phase of 17 degrees, also just after peak. Two independent datasets, different periods, different curators, same direction.

The superposed-epoch picture sharpened it:

![Figure 2. Superposed-epoch analysis around 18 solar maxima: conflict onsets peak at or just after solar max.](https://gist.githubusercontent.com/Macaulay001/f59401e6fd96a6d5279ca1a54db1e81c/raw/fig2_superposed.png)
*Figure 2. Superposed-epoch analysis around 18 solar maxima: conflict onsets peak at or just after solar max.*


*Figure 2. Superposed-epoch analysis around 18 solar maxima: conflict onsets peak at or just after solar max.*


*Figure 2. Superposed-epoch analysis around 18 solar maxima: conflict onsets peak at or just after solar max.*


| Lag from solar max (years) | Mean CoW onsets / year |
|---|---|
| -5 | 1.44 |
| -3 | 1.50 |
| -1 | 2.06 |
| 0 (peak) | 3.06 |
| +1 | 2.39 |
| +3 | 1.94 |
| +5 | 1.78 |

Roughly twice as many onsets at the peak as at the trough. Not the 60 to 80 percent enrichment Chizhevsky claimed. Closer to a factor of two.

Granger causality, sunspot to CoW onsets at lag 2, came in at F = 2.56, p = 0.08. Marginal, not significant by conventional thresholds. I would not lean on it.

So the result, on this sample, is mixed in an interesting way. The standard linear correlation (Pearson r) is weak and non-significant. The cycle-phase test is highly non-uniform and points to the same phase in two independent datasets. The mass excess at peak versus minimum is about 2x, not the dramatic clustering Chizhevsky reported.

A few notes on what could be wrong. Twentieth-century events are over-reported in both CoW and UCDP, and the twentieth century also happens to contain the strongest solar cycles (19 and 21 and 22). That coincidence alone could inflate the phase test. I tried to control for it with the 31-year running-mean detrend before the Pearson test, which is why r drops to +0.13. But the phase test is run on raw onsets, and a clean way to detrend a point process in phase space is not obvious. I tried restricting to 1816 to 1945 only (cuts the modern bulge) and the Rayleigh p was still 0.012 for CoW. That suggests the phase clustering is not purely a 20th-century artifact, but I would not stake a lot on a single subsample.

The 18 peaks are not independent in the strong statistical sense. They are spaced by roughly eleven years. The effective sample size for the superposed-epoch test is closer to 18 than to the 190 raw years.

## What it may mean

The honest read: Chizhevsky's qualitative claim, that mass-conflict onsets cluster non-uniformly within the solar cycle and tend to fall near the peak, survives a reasonably careful test on two modern datasets he never saw. It survives at a smaller effect size than he reported. It does not survive as a clean linear correlation, and it does not pass a strict Granger test.

I am not claiming the sun causes wars. I want to repeat that because the topic attracts people who do want to claim it. The phase pattern is consistent with a statistical association whose mechanism is entirely unknown to me. Possibilities I cannot rule out: a confound with century-scale modernization that I detrended imperfectly, a Type-I error inflated by the non-independence of solar cycles, an indirect climate or agricultural pathway (solar UV and harvest failure has at least a plausible story), or some neurochemical-geomagnetic effect of the kind Persinger and Halberg wrote about. I tested none of these. I tested whether a phase signal exists.

What I think this places heliobiology in is a category I want to call "not refuted as statistical association, not confirmed as causal mechanism." That is different from astrology, where neither half holds. It is also different from a real scientific theory, where mechanism is identified. It sits in the awkward middle, and the awkward middle is where I think most forgotten claims actually live.

If the phase signal is real and not an artifact, it would not be a breakthrough so much as a small empirical anomaly that mainstream conflict studies has been incurious about for a century. That is interesting on its own.

## Loose ends

I did not analyze events before 1816 because CoW does not cover them, and using LLM-extracted Wikipedia chronologies for pre-1816 events would mix observational and curatorial bias in ways I have not figured out how to handle. Chizhevsky's original index ran back to roughly 1600 and that is where the strongest part of his claim lives. With another week I would build a clean pre-1816 series from Wikidata, run the same phase test on it, and see whether the signal holds or vanishes. I would also try wavelet coherence between sunspot and onset rate at the 11-year band, which I skipped. If anyone has a vetted historical event series that predates CoW and was not assembled with this question in mind, I would like to hear about it.
