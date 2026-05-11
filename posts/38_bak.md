# I dropped 140,000 grains onto a virtual sandpile. Avalanches obeyed a power law across four decades.

*Part 38 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1987 Per Bak, Chao Tang, and Kurt Wiesenfeld pushed a four-page paper into Physical Review Letters with a title that promised more than the math could carry: "Self-Organized Criticality: An Explanation of 1/f Noise". The setup was a toy. Imagine a square grid where each cell holds a non-negative integer height. Drop a grain on a random cell. If any cell hits height 4 or more, it topples, losing 4 grains and giving one to each of its four neighbors. Toppling can trigger more toppling. When the cascade ends, drop another grain. That is the entire model.

What made the paper notorious was the claim attached to it. The authors said that this system, with no tuned parameter and no external scale, would settle on its own into a state where avalanche sizes followed a power-law distribution. The grain-by-grain driving was infinitesimally slow compared with the toppling, and that separation of timescales was enough. The system would find criticality without anyone aiming it there. Bak called it self-organized criticality (SOC) and spent the next fifteen years arguing that earthquakes, forest fires, neuronal cascades, and financial crashes all worked this way. The community largely accepted the sandpile picture, then argued for years about whether it really mapped onto any of those phenomena, then quietly stopped citing the original paper while still using its vocabulary.

I had quoted the SOC story in two graduate seminars without ever running the simulation. The model is simple enough that it lives in a single page of Python. I had also never checked Bak's specific number for the avalanche-size exponent, which he reported as tau ~ 1.0. Modern simulations put it closer to 1.27. That is not a small disagreement on a log-log plot.

## What I tried

The plan was to run the canonical BTW sandpile at two sizes, 50x50 and 100x100, with open boundaries (grains that try to leave the edge just fall off, which is what dissipates the system and lets it reach a stationary regime). At each time step I picked a random site, added one grain, and ran the toppling rule to completion before adding the next grain. I recorded two things for every drop: the avalanche size (number of toppling events triggered) and the activity time series (toppling events per micro-step of the relaxation, treating the whole run as a single sequence). The first feeds the power-law fit. The second feeds the power-spectrum fit, which is where the 1/f noise claim lives.

Warmup mattered. A freshly empty lattice sits below threshold for a long time and produces no avalanches at all. I dropped 5,000 to 8,000 grains before starting to record, which is more than enough to fill the average height to its stationary value (around 2.1 grains per cell on a square lattice with open boundaries) and start generating cascades that reach the boundary.

For the toppling step I used a parallel update. At each micro-step I found every cell with height greater than or equal to 4, decremented each by 4, and added 1 to each of its four neighbors (with edge cells simply losing their boundary contribution to the void). The order of toppling does not matter for the final configuration (that is the "abelian" property which gave the sandpile its other name), but it does affect the micro-time series, so the spectrum I report is conditional on parallel updates.

For the power-law fit I used the maximum-likelihood estimator for a continuous power law with a lower cutoff s_min, which is the only honest thing to do with heavy-tailed data: histogram slopes are biased and unreliable. The estimator is tau = 1 + n / sum(log(s / (s_min - 0.5))), with standard error (tau - 1) / sqrt(n).

```python
# core BTW step: parallel toppling with open boundary
while True:
    mask = h >= 4
    n_top = int(mask.sum())
    if n_top == 0:
        break
    avalanche += n_top
    h[mask] -= 4
    h[:-1, :] += mask[1:, :]   # up
    h[1:, :]  += mask[:-1, :]  # down
    h[:, :-1] += mask[:, 1:]   # left
    h[:, 1:]  += mask[:, :-1]  # right
```

That is the entire dynamics. The rest of the script is bookkeeping.

## What happened

The 100x100 grid ran 60,000 drops in about three minutes. Of those drops, 23,000 produced at least one toppling. The largest single avalanche knocked over 45,311 cells, which is more than four times the lattice area, because a single cell can topple many times in one cascade. Mean avalanche size was 738.

On a log-log histogram of avalanche sizes, the distribution looks like a line. Not a fuzzy line, a line. I had read about SOC for a decade and had never actually seen the plot from my own data, and the cleanliness of it caught me by surprise.

![Avalanche size distribution on 50x50 and 100x100 BTW sandpiles, log-binned, with reference slopes at tau=1.0 and tau=1.27.](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/38_avalanche_distribution.png)

The MLE exponents:

| grid | n drops | mean size | max size | tau |
|------|---------|-----------|----------|-----|
| 50x50 | 80,000 | 220 | 10,067 | 1.347 +/- 0.002 |
| 100x100 | 60,000 | 739 | 45,311 | 1.299 +/- 0.002 |

Two things to notice. The exponent drifted downward as the grid got bigger, which is what theory predicts for a system with multi-fractal corrections (different moments of the size distribution scale with different exponents, so the apparent tau depends on the range you fit). The 100x100 value of 1.30 sits within the published range of 1.27 to 1.30 from much larger simulations by Lubeck (1997) and De Menech (1998). Bak's original 1.0 was a quick-look fit on a short tail. It seems to have been wrong from the start, though the qualitative claim that the distribution is power-law was right.

The cutoff scaled with system size roughly as L^2.2, in the right ballpark for the predicted finite-size scaling. There was no characteristic avalanche size; the distribution simply ran out at the lattice boundary.

![Power spectrum of toppling activity on both grids, with linear log-log fit between 1e-3 and 0.1 cycles per micro-step.](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/38_power_spectrum.png)

The power spectrum told a more nuanced story. Both grids showed scale-free behavior over three decades of frequency. A linear fit on log-log in the band 1e-3 to 0.1 gave beta = 1.58 at L=50 and beta = 1.65 at L=100. Scale-free, yes. Pure 1/f, no. Henrik Jensen pointed this out in 1989, a couple of years after the original paper, and the BTW group conceded the point: the activity spectrum is closer to 1/f^2 than to 1/f, which means it looks more like Brownian motion than like the famous flicker noise in resistors and quasar fluxes. The "explanation of 1/f noise" in the original title may have been the part that aged worst.

## What it may mean

If you accept that the cleanest demonstration of self-organized criticality produces avalanches with tau in the 1.27 to 1.35 range and a spectrum with beta near 1.6, then the model still does the heavy lifting it was originally hired for. There is no parameter to tune. The slow grain-by-grain driving and the local threshold rule are enough. The system finds a critical point on its own.

That matters when you ask whether real-world heavy-tailed phenomena need SOC as an explanation. Earthquakes follow the Gutenberg-Richter law, which is a power law with an exponent near 1 for cumulative magnitudes, equivalent to about 1.7 in size, depending on definition. That number is close to what the sandpile gives, but not identical, and seismologists have spent decades pointing out that the analogy is loose. Forest fires obey a power law with exponent near 1.4 in burn area, also in the SOC ballpark. Neural avalanches in cortical slice cultures, measured by Beggs and Plenz in 2003, came in at tau = 1.5 for size, which is close but slightly steeper.

These numbers are similar enough to make SOC tempting as a universal explanation and different enough to make any specific mapping awkward. What the sandpile may really be telling us is something weaker but still useful: that systems with slow driving, local thresholds, and a way to dissipate energy at the boundary do not need a tuned mechanism to produce heavy tails. They get heavy tails for free. Whether a given heavy tail in nature comes from this mechanism or another is an empirical question, not a theorem.

The thing that struck me most was not the exponent. It was the absence of any characteristic scale on the histogram. A single drop can do nothing, or it can topple half the lattice. You cannot predict which from looking at the configuration alone, because the configuration is statistically stationary and the future is set by which random cell receives the next grain. That is a hard fact about a deterministic toppling rule. The world appears to keep this kind of indeterminacy on the menu.

## Loose ends

One run, one update rule, two grid sizes. The published literature uses lattices ten times bigger and averages over many independent runs to push the power-law range out further. The spectrum fit depends on which frequency band you pick; widening it pulls beta up. I did not check the non-abelian Manna model or the directed Oslo rice pile, both of which give different exponents and live in different universality classes. With another week I would run 500x500 and use proper Welch averaging on the spectrum so the beta number stops drifting with my window choice. The code runs in a few minutes per grid, so anyone with a laptop can reproduce this.
