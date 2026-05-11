# I simulated Maynard Smith's 1973 Hawk-Dove game. The mixed equilibrium showed up at p = 0.40, exactly where he said.

*Part 27 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1973, John Maynard Smith and George Price published a short Nature paper called "The Logic of Animal Conflict". They wanted to know why animals so rarely fight to the death. A red deer with antlers can in principle gore another red deer, but usually the two of them shove each other for a while and one walks away. Why?

Their answer was a game. Imagine two strategies in a population. Hawks always escalate. Doves always display and retreat if the opponent escalates. The payoff for winning a contest is V. The cost of losing an escalated fight is C. If you work out the expected payoffs, a population of all Hawks is invadable by Doves whenever C is bigger than V, because Hawks keep paying the cost against each other. A population of all Doves is invadable by Hawks too, because a lone Hawk wins every contest. The stable point is a mix, with Hawk frequency p* = V / C.

The idea became the founding example of an Evolutionarily Stable Strategy. It is taught in every behavioural ecology course. It is also fifty-three years old, simple enough to simulate in an afternoon, and worth checking that the prediction actually holds when you let agents adapt by replication and imitation rather than by handwaving.

## What I tried

The plan had three parts.

Part one was the textbook check. Run continuous replicator dynamics on the Hawk-Dove payoff matrix with V = 2 and C = 5, which gives a predicted mixed equilibrium at p* = V/C = 0.4. Start from five different initial frequencies and see whether they all flow to the same place. This is the deterministic version of evolution: the strategy that does better than the population average grows in proportion.

Part two was the noisy version. A real population is finite and stochastic. I used a Moran-style update where two random individuals are picked each step, the lower-payoff one imitates the higher-payoff one with a probability set by a logistic function of the payoff gap, and a small mutation rate flips strategies at rate 0.002 per step. I ran this for N = 50, 200, and 1000 individuals, 40,000 steps each, and asked whether the mean Hawk frequency tracks p* and whether the variance scales with population size.

Part three was the degenerate case. ESS theory predicts that when C < V (escalation is cheap), there is no mixed equilibrium and Hawks should fixate. I set V = 5, C = 2, and ran the deterministic dynamics from three initial conditions.

The whole thing is about 150 lines of numpy. No external simulators, no calibration, no fitting. The payoff matrix and the update rule are the only inputs.

## What happened

Phase one was clean. Every initial condition (0.05, 0.2, 0.5, 0.8, 0.95) converged to p = 0.4000 within numerical precision after 2000 time steps. The trajectories curve toward the dashed line at V/C and stick there.

The core of the replicator step is short.

```python
def payoff_hawk(p, V, C):
    return p * (V - C) / 2.0 + (1 - p) * V

def payoff_dove(p, V, C):
    return (1 - p) * V / 2.0

def replicator_step(p, V, C, dt):
    fh = payoff_hawk(p, V, C)
    fbar = p * fh + (1 - p) * payoff_dove(p, V, C)
    return p + dt * p * (fh - fbar)
```

Phase two was the part I was less sure about. Imitation dynamics are not strict Moran, and small populations can drift far from the deterministic prediction. Here is what came out.

| N    | mean p | sd p   |
|------|--------|--------|
| 50   | 0.351  | 0.186  |
| 200  | 0.414  | 0.062  |
| 1000 | 0.407  | 0.032  |

At N = 1000 the mean is within 2% of p* = 0.4 and the standard deviation is about 0.03. At N = 50 the mean drifts low (0.351) and the standard deviation is six times bigger. The 1/N scaling for variance is rough but visible: going from N = 200 to N = 1000 drops the variance by a factor of about 3.8, which is in the same ballpark as the expected factor of 5. The small-N bias toward fewer Hawks is interesting. It may reflect that Hawk-Hawk fights are absorbing in the sense that when the Hawk minority shrinks, the remaining Hawks fight each other and lose payoff faster than Doves do. I have not proved this; it is a guess from staring at the trace.

Phase three was the sanity check. With V = 5 and C = 2, the formula gives V/C = 2.5, which is not a valid frequency. The theory says no mixed ESS exists and Hawks should fixate. They did. All three trajectories from p0 in (0.05, 0.2, 0.5) ran up to p = 1.0 within 500 steps.

So on this simulation, the 1973 prediction holds in all three regimes I tested. Deterministic convergence is exact. Noisy convergence is approximate and degraded at small N. The boundary case fixates as predicted.

## What it may mean

The Hawk-Dove result is not contested in any serious way, so this is more of an audit than a discovery. The interesting bit is the small-N behaviour. Modal behaviour in groups of 50 or fewer (which is the size of many real animal social groups, primate troops, fish schools, human bands before agriculture) may not track the deterministic ESS very well. The mean Hawk frequency in this run sat at 0.35 instead of 0.40, and individual time windows could spend long stretches at 0.2 or 0.55. If you tried to infer V/C from a small population's observed Hawk frequency, you could be off by 15% or more before any measurement noise enters.

The other thing worth flagging is that the imitation rule matters. I picked a logistic-payoff-difference rule with a moderate slope. Other rules (pairwise comparison with a steep slope, or strict Moran with fitness-proportional reproduction) shift the small-N bias and the variance scaling. The deterministic limit is the same, but the route there depends on the microscopic dynamics. For a 1973 paper that did not specify the microscopic dynamics, this is fine. For a modern attempt to fit ESS predictions to behavioural data, it matters which update rule you assume.

I will not claim this proves anything beyond the math. It is a reproduction. But it is a fast reproduction, and the agreement is sharp enough that the prediction probably deserves its place in textbooks.

## Loose ends

What I did not test: asymmetric Hawk-Dove with an ownership convention (the Bourgeois strategy that wins in real animal contests), continuous-strategy versions where the level of escalation is a real number, and finite-memory agents that condition on the previous opponent. Each of these is one more afternoon. The other open question is whether the small-N bias I observed (mean 0.351 instead of 0.400 at N = 50) has a clean analytic form. The Fokker-Planck approximation for imitation dynamics should give it; I have not done the calculation. If anyone has done this for the Hawk-Dove case with logistic imitation, please point me at the reference.

You can reproduce the whole thing in about thirty seconds on a laptop, with one file and numpy.
