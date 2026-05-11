# I tested the 1772 Titius-Bode law on the Solar System and 5 exoplanet systems. Three of six beat random spacing at p < 0.01.

*Part 109 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1766 Johann Daniel Titius slipped a footnote into a German translation of Charles Bonnet's *Contemplation de la Nature*. He noticed that if you write down the sequence 0, 3, 6, 12, 24, 48, 96, 192, add 4 to each term, and divide by 10, you get a list of numbers that match the known planetary distances in astronomical units, with one number (2.8 AU, between Mars and Jupiter) belonging to no known planet. Johann Elert Bode promoted the formula in 1772 and it has carried his name ever since.

The closed form is `a = 0.4 + 0.3 * 2^n`. For Mercury you take the limit n -> -infinity so a -> 0.4. For Venus through Saturn the rule lands within a few percent of the actual semi-major axes. The 2.8 AU gap was the headline prediction: Piazzi found Ceres at 2.77 AU in 1801, and Herschel had already found Uranus in 1781 at 19.2 AU, about 2% from the Bode value of 19.6. For half a century the law looked like a discovery.

Then Neptune was found in 1846 at 30.1 AU. Bode's rule said 38.8. A 29% miss is not a rounding error, and astronomers have argued about what the law means ever since. The modern consensus, roughly, is that some geometric spacing is real, driven by orbital stability constraints, but the specific Titius-Bode coefficients are coincidence. Hayes and Tremaine in 1998 (*Icarus* 135) showed that random orbits drawn from a no-crossing prior fit a Bode-like law about as well as the real planets do. Lynch 2003 (*MNRAS* 341) put it more bluntly and called the law numerology. But every few years a new exoplanet result revives the argument, most recently Bovaird and Lineweaver in 2013 finding Bode-style spacing in many Kepler multi-planet systems.

I wanted to run the comparison myself with the simplest possible control.

## What I tried

Three tests. First, the classical formula on the 9 Solar-System bodies that have been used in the historical argument (Mercury, Venus, Earth, Mars, Ceres, Jupiter, Saturn, Uranus, Neptune). Compute the residual at each body and the RMS percentage error across all nine.

Second, fit a generalized geometric law `a_n = a_0 * C^n` to five exoplanet systems that have at least five planets each: Kepler-11 (six planets), Kepler-90 (eight), TRAPPIST-1 (seven), HD 10180 (six), and 55 Cancri (five). I pulled the semi-major axes from the NASA Exoplanet Archive median values. The geometric fit reduces to a straight-line fit in `(n, log a)`, so the residual is well defined.

Third, the part I cared about. For each system of K planets, draw K-2 intermediate semi-major axes from a log-uniform distribution between the observed innermost and outermost planet. This is the laziest reasonable null: it preserves the system's size and number of planets and asks whether the actual ordering of internal planets is more "geometric" than random log-spacing would produce. Repeat 10000 times per system, refit the geometric law to each random draw, record the RMS log10 residual. Compare the real system's RMS to the random distribution.

I deliberately did not use a Hill-stable null. The Hill criterion (Hayes and Tremaine's choice) builds in a no-crossing constraint that already imposes some geometric spacing, so a real system has to beat that floor to look special. Log-uniform is more permissive. If a real system still beats log-uniform random spacings, that's a stronger signal than beating Hill-stable spacings.

## What happened

The classical Bode law on the Solar System has an RMS percent error of 10.1% across the nine bodies, dominated by Neptune's 29% miss. Drop Neptune and the RMS error falls to 3.1%, which is the number Bode supporters in 1846 were defending. Pluto's semi-major axis at 39.5 AU is within 2% of the Bode prediction of 38.8, which is why some 20th-century writers preferred "Pluto is the n=7 body and Neptune migrated inward".

Here is the core of the geometric fit and the control:

```python
def fit_geometric(a):
    n = np.arange(len(a))
    y = np.log10(np.asarray(a))
    slope, intercept = np.polyfit(n, y, 1)
    resid = y - (slope * n + intercept)
    return 10**intercept, 10**slope, np.sqrt((resid**2).mean())

def random_log_uniform_seq(K, a_min, a_max, rng):
    inner = np.exp(rng.uniform(np.log(a_min), np.log(a_max), size=K - 2))
    return np.sort(np.concatenate([[a_min], inner, [a_max]]))

real_rms = fit_geometric(a)[2]
rand_rms = [fit_geometric(random_log_uniform_seq(K, a[0], a[-1], rng))[2]
            for _ in range(10000)]
p = (np.array(rand_rms) <= real_rms).mean()
```

The numbers that matter, sorted by p-value:

| System      | K | RMS log10 | P(random <= real) |
|-------------|---|-----------|-------------------|
| Solar       | 9 | 0.050     | 0.001             |
| TRAPPIST-1  | 7 | 0.018     | 0.002             |
| HD 10180    | 6 | 0.053     | 0.007             |
| 55 Cancri   | 5 | 0.135     | 0.071             |
| Kepler-11   | 6 | 0.046     | 0.154             |
| Kepler-90   | 8 | 0.088     | 0.402             |

Three of six beat random log-uniform spacing at p < 0.01. Two are essentially indistinguishable from random. The split surprised me. TRAPPIST-1 has the cleanest geometric fit (RMS log10 = 0.018), which is consistent with the known resonance chain locking those seven planets into integer period ratios. The Solar System sits at p = 0.001 even with the Neptune problem, because the random draws frequently produce worse Neptune-analog residuals than the real Solar System does.

Kepler-90 is the most "compressed plus extended" system in the set, with four planets crowded inside 0.32 AU and four spread out to 1 AU. A two-parameter geometric law fits it badly (RMS = 0.088), and random log-uniform draws fit about as badly, so the comparison is uninformative.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/109_real_vs_random_controls.png)

The control figure makes the picture concrete. For each system the gray bars are the 5-95% range of random log-uniform RMS values, and the red dot is the real system. Solar, TRAPPIST-1, and HD 10180 sit clearly below their gray bars. Kepler-11 and Kepler-90 sit inside them.

## What it may mean

Two readings, both partial.

The Bode-friendly reading: geometric spacing is real in some systems and the 1772 rule was capturing a constraint that operates in planetary disks. TRAPPIST-1's resonance chain gives a physical reason: mean-motion resonances stabilize integer period ratios, and integer period ratios produce geometric semi-major-axis ratios via Kepler's third law. The Solar System may have settled into a similar (looser) resonance structure during the planet-migration phase that the Nice model describes.

The Bode-skeptical reading: a two-parameter fit can do a lot. Any monotone sequence of K = 5 to 8 numbers can be fit by `a_0 * C^n` to within a factor of two, and matching against random log-uniform draws is a weak null because both the data and the null are smooth. The systems where Bode beats random by p < 0.01 are the systems with known resonance chains or near-resonance configurations. The systems where it doesn't are the ones where individual planet detections came with the largest fractional uncertainties in semi-major axis.

The honest position sits between. The classical 0.4 + 0.3 * 2^n is too specific to be deep, and the Neptune failure is real. A generalized geometric law captures something true about how some multi-planet systems pack their orbits, and the Solar System is firmly in that group on this sample, though I'd want twenty more systems before I'd argue the point in a journal.

## Loose ends

Three things would sharpen this. First, switch the null from log-uniform to Hill-stable, which is the harder test and the one Hayes and Tremaine ran. Second, extend the exoplanet set to all confirmed 5+ planet systems (there are around 15 now) and report the fraction that beat random at p < 0.01 with a Bonferroni correction. Third, separate the resonance-chain systems from the rest before pooling, because TRAPPIST-1 has different physics from HD 10180. With a week and the NASA archive query interface this is an afternoon's work.
