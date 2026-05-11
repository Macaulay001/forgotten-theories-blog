# Hutchinson's paradox of the plankton, 64 years on, in 200 lines of Python

*Part 20 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1961, G. Evelyn Hutchinson published a short, slightly puzzled paper in *The American Naturalist* (volume 95, page 137) titled "The paradox of the plankton." The puzzle was simple. Gause's principle of competitive exclusion, well established by then, said that two species competing for the same limiting resource cannot coexist indefinitely; one must win. And yet a single bucket of seawater, drawn from what looked like a homogeneous, well mixed water column, contained dozens of phytoplankton species, all apparently competing for the same handful of nutrients. Light, nitrogen, phosphorus, silica. That was about it.

Hutchinson floated three possible escapes. First, environmental conditions fluctuate fast enough that no single competitor reaches equilibrium dominance. Second, the water column is not really homogeneous at the scales the plankton experience. Third, the system is open, with immigration topping up losers before they vanish. He preferred the first.

I wanted to see how well that first answer actually holds up if you simulate it.

## What I tried

The cleanest test is Lotka-Volterra competition. You set up N species sharing a resource pool, you write down the standard equations, and you let them run. If Gause is right, in a constant environment you end up with one or two survivors. If Hutchinson's first resolution is right, switching the resource availability up and down over time should let many more species coexist.

I wrote a small simulator with thirty species, a random competition matrix (diagonal one, off-diagonal uniform on [0.5, 1.5]), and integration time T=2000 with dt=0.05. Ten random seeds per condition. I tested three modes. The first was constant carrying capacity, which is the boring control. The second was 5% multiplicative noise on the growth, basically demographic stochasticity without any environmental signal. The third was the Hutchinson mode: a slow sinusoidal modulation of resource availability with broadband noise on top, varying per species. I also extended one fluctuating run to T=5000 to check whether the surviving set keeps eroding or stabilizes.

Then I compared the simulation output against a real number. Sunagawa et al. (2015, *Science* 348:1261359) reported 1500 to 3500 plankton OTUs per single sample across the Tara Oceans transects, depending on latitude. That is the number Hutchinson's resolution has to explain, more or less. Not dozens. Thousands.

The core of the simulation is genuinely tiny:

```python
# Lotka-Volterra step with optional resource fluctuation
def step(x, r, A, dt=0.05, noise=0.0, rng=None):
    dx = x * (r - A @ x)
    if noise > 0:
        dx += noise * x * rng.normal(0, 1, size=x.shape)
    return np.maximum(x + dt * dx, 0)

# Hutchinson mode: slow sine + broadband noise on r
r_t = r * (1 + 0.4 * np.sin(t * dt * 0.3) +
           0.2 * rng.normal(0, 1, size=N))
```

A species was counted as surviving if its abundance was still above 1% of carrying capacity at the end of the run.

## What happened

The numbers were not what Hutchinson would have wanted, although they were not catastrophic for him either. Out of thirty starting species, the average surviving count came in like this:

| Mode | Mean survivors (of 30) |
|------|-----------------------:|
| Constant environment | 3.7 |
| 5% stochastic noise | 3.2 |
| Fluctuating resources (Hutchinson 1) | 3.4 |
| Fluctuating, extended T=5000 | 4 |

Several things stand out. The constant-environment number is not one. Random competition matrices with off-diagonals that overlap one (a common case here) admit a few coexisting species even without any environmental help, because some species pairs sit in a stable interior equilibrium by accident. So Gause-style strict exclusion is already a simplification. Fine. But the bigger surprise was that flipping on fluctuating resources moved the average from 3.7 down to 3.4. Adding noise did not help. If anything it shaved off about half a species.

I let one of the fluctuating runs go out to T=5000 to check that I was not just measuring transients. Four species. About the same. The fluctuation does not nucleate new coexistence; it mostly reshuffles which of the few stable coexisting clusters you land in.

To be fair to Hutchinson, I also implemented a Chesson-style storage effect variant in a separate file. The idea is to give species long-lived dormant stages so a species that gets clobbered during a bad epoch can wait it out and re-emerge when conditions swing back. That helps a bit. You can get the survivor count above ten with aggressive enough storage and the right autocorrelation structure on the environment. I could not get it past roughly fifteen on a thirty-species ensemble without making the parameters frankly silly.

Fifteen out of thirty is not 3500.

The gap between what a single-mechanism Hutchinson model can produce and what Tara Oceans actually sees is at least two orders of magnitude. That is the part I find interesting. It does not mean Hutchinson was wrong about fluctuation mattering; it means fluctuation in a well-mixed bulk water column, on its own, cannot do the heavy lifting. Something else has to be carrying most of the diversity.

I went back to the literature to read what people who have spent careers on this think. Chesson (1994, *Annu Rev Ecol Syst*) is the cleanest formal statement. His decomposition splits coexistence into a few terms: the storage effect (basically Hutchinson 1 plus dormancy), relative nonlinearity of competition, and resource partitioning. In Chesson's framing, fluctuation by itself is not a mechanism. It only becomes one when paired with species-specific responses and overlapping generations. That maps onto what I saw. My naïve fluctuation, applied identically to all thirty species in a fully mixed pool, just shakes the whole system; it does not give any one species an asynchronous refuge.

There is also a separate strand of work on micro-scale spatial heterogeneity. Turbulent mixing creates transient patches at the centimeter scale, and a plankton cell's relevant Reynolds number means its world is locally not homogeneous at all. Multi-resource niche partitioning, between nitrogen, phosphorus, silica, iron, light wavelengths, also opens up more niche dimensions than a single-resource model captures. And then there is predation, viruses and grazers, which can keep would-be dominants in check long enough for everyone else to survive.

## What it may mean

If I had to summarize what this small experiment may say, it is that Hutchinson identified a real puzzle but undersold its difficulty. The single mechanism he preferred appears insufficient in a simple LV simulation, and it stays insufficient even after you bolt on a storage term. The qualitative direction is correct, fluctuation does help, but the quantitative gap to real-ocean richness is too large for any one mechanism to bridge on its own.

This is not a proof. The Lotka-Volterra model is a crude caricature, and the survivor count depends on how I define survival, the variance of the competition matrix, and the timescale ratio between fluctuation and growth. With friendlier parameter choices the fluctuating mode could plausibly support more species. But the order-of-magnitude gap is robust enough across the seeds I ran that I do not think any reasonable tuning closes it.

The takeaway, to the extent I trust it, is the Chesson view: paradox-level diversity in nature seems to need several mechanisms operating at once, and the relative contribution of each is still a live debate. Storage effect plus spatial heterogeneity plus multi-resource niches plus top-down predation. Pick any one, and the math tells you it is not enough.

## Loose ends

The honest version of this experiment would skip simulation and go directly to the Tara Oceans OTU tables, fit a Chesson decomposition to the empirical abundance distributions, and ask which term carries how much variance. I did not do that here. A week of work and a careful look at the metabarcoding pipeline would get most of the way. If anyone has already done that decomposition on Tara Oceans data and wants to compare notes, I would happily read it.
