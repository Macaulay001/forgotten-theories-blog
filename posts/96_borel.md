# I sampled a million coin flips with decaying probabilities. The 1909 dividing line showed up exactly where Borel said it would.

*Part 96 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1909 Émile Borel was working through a problem about decimal expansions of real numbers, and he proved a small lemma to dispatch a side case. Eight years later, Francesco Paolo Cantelli sharpened it. Together the two halves became one of the cleanest dichotomies in probability theory. Given a sequence of events A_1, A_2, A_3, ..., look at the sum of their probabilities. If the sum is finite, then with probability one only finitely many of the events happen. If the events are independent and the sum is infinite, then with probability one infinitely many of them happen. That is the whole statement.

The result is not forgotten, exactly. It is in every graduate textbook. What is forgotten is how counterintuitive the boundary actually feels until you watch it on a computer. Two sequences can both have probabilities decaying to zero, and one of them gives a stream that never stops, and the other gives a handful of events and then silence forever. The dividing line is whether Σ P(A_n) is finite. Nothing else matters.

I wanted to see that line.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/96_cumulative_counts.png)

## What I tried

The simplest pair that straddles the boundary is the harmonic series and its square. Take p_n = 1/n: the sum 1 + 1/2 + 1/3 + ... diverges. Take p_n = 1/n^2: the sum is π²/6 ≈ 1.6449, a constant Euler nailed down in 1734. Both probabilities go to zero as n grows, but the lemmas say their long-run behavior could not be more different.

The plan was almost embarrassingly direct. For each n from 1 to N = 10^6, draw an independent uniform u_n in [0, 1) and record a hit if u_n < p_n. Keep a running count. Do this 1000 times for each probability sequence so I can look at the distribution rather than a single sample path. Then check the curves against two textbook predictions. For p_n = 1/n, the expected cumulative count up to N is the N-th harmonic number, which behaves like log N + γ where γ ≈ 0.5772 is the Euler-Mascheroni constant. For p_n = 1/n², the expected count converges to π²/6 from below, so the curves should bend over and stop.

I added a third piece. The monkey-and-typewriter argument is the most famous application of the second lemma. A typing monkey produces independent characters; the probability of a target phrase in any given window is positive but tiny. The second lemma says that phrase will occur infinitely often, with probability one. I picked a tractable analogue: a four-letter target with p = 26^-4 ≈ 2.19 × 10⁻⁶, and ran 5 × 10^6 independent trials per replicate across 200 replicates. The expected number of hits per replicate is about 11. The question is whether any replicate gets zero.

A single chunk of code is the whole experiment:

```python
N = 1_000_000
trials = 1000
n = np.arange(1, N + 1, dtype=np.float64)
p_div = 1.0 / n
p_conv = 1.0 / n**2

def simulate(p):
    out = np.zeros((trials, len(snap_idx)), dtype=np.int64)
    for s in range(0, trials, 50):
        u = rng.random((50, len(p)))
        hits = (u < p).astype(np.int8)
        out[s:s+50] = np.cumsum(hits, axis=1)[:, snap_idx - 1]
    return out
```

The whole simulation ran in about three minutes on one machine.

## What happened

The divergent and convergent cases split exactly as the lemmas predict.

For p_n = 1/n, the median cumulative count at N = 10^6 across 1000 trials was 14, and the mean was 14.62. The theoretical mean is log(10^6) + γ ≈ 14.39. None of the 1000 trials had fewer than 6 hits. The growth is slow but unmistakably steady on a log axis. If I doubled N to 2 × 10^6, I would expect about 0.69 more hits per trial. Push N to 10^9, expected count rises to roughly 21. The series never stops contributing.

For p_n = 1/n², the median count at N = 10^6 was 2, and the mean was 1.668. The theoretical mean is π²/6 ≈ 1.6449. The largest cumulative count seen across all 1000 trials was 4. Most of the action happened in the first few hundred draws; after roughly n = 1000 the trials sat where they were. Nobody got rich rolling on this sequence.

A small comparison table:

| Sequence | mean count at N=10^6 | theory | max across 1000 trials |
|---|---|---|---|
| p_n = 1/n (Σ diverges) | 14.62 | log N + γ ≈ 14.39 | 26 |
| p_n = 1/n² (Σ converges) | 1.668 | π²/6 ≈ 1.6449 | 4 |

For the monkeys, the 200 replicates produced a mean of 11.49 hits with standard deviation 3.26, close to the Poisson prediction of mean 10.94 and std 3.31. Crucially, zero replicates had zero hits. At a per-trial probability of 2 in a million, five million independent trials reliably caught the target. The second lemma's "almost surely infinitely often" is the limiting statement of what is already a very strong tendency at finite scale.

The two halves of the dichotomy show different fingerprints in the figure. The divergent curve drifts upward in a logarithmic creep, with the 5-95% band widening as N grows. The convergent curve runs into a ceiling, with a band that locks in by n ≈ 10^4 and never moves again. The transition is not gradual. It is the difference between a series that contributes at every scale and one that contributes a finite total budget that gets spent early.

## What it may mean

The Borel-Cantelli lemmas are doing a quiet job underneath most of the probabilistic results people quote. Kolmogorov's 0-1 law leans on them. The Strong Law of Large Numbers follows in a few lines from the first half. Recurrence theorems for random walks invoke the second half. The monkey-and-typewriter argument that gets trotted out in popular books is exactly the second lemma applied to independent character streams.

The intuition the simulation gave me is this. Probabilities decaying to zero are not enough information to know what happens in the long run. You have to know how fast they decay. The harmonic boundary at 1/n versus 1/n² is the same boundary that separates "eventually" from "never again". On either side, the long-run behavior is essentially deterministic, in the sense that you get the same answer almost surely on every sample path.

This is not a proof of anything new. It is a visualization of a 1909 statement that turns out to be sharp at the level of arithmetic. What I found compelling is that 1000 independent runs all behaved within the band, and none of them violated the dichotomy. The lemmas appear robust at this sample size.

## Loose ends

I did not test the case where the second lemma fails: pairwise independence is not enough. Constructions like Erdős-Rényi pairwise events can satisfy Σ P(A_n) = ∞ and still have P(A_n i.o.) < 1, and seeing one of those simulated would make the independence hypothesis feel less like a footnote. With another day I would also push the divergent simulation to N = 10^9 to watch log-scale growth continue, and try a fractional case like p_n = 1/(n log² n) which sits on the convergent side of the boundary with much slower decay. The replication code is short, and anyone with a laptop can rerun it in under five minutes.
