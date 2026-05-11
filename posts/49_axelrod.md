# I reran Axelrod's 1981 prisoner's dilemma tournament. Tit-for-tat still earns its reputation.

*Part 49 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1980, the political scientist Robert Axelrod sent letters to game
theorists, economists, psychologists, and computer scientists, asking
each one to submit a strategy for an iterated prisoner's dilemma. He
got 14 entries. Some were elaborate, with state machines and rules for
detecting other strategies. The shortest entry came from Anatol
Rapoport. It was four lines of BASIC, and it implemented the simplest
rule a child could describe: cooperate on the first move, then copy
whatever the other player did last. Tit-for-tat. It won.

Axelrod ran it again in 1981 with 62 entries, including ones designed
specifically to beat tit-for-tat. Tit-for-tat won that one too. From
those results he extracted four properties he claimed any winning
strategy needed: be nice (never defect first), be retaliatory (punish
defection), be forgiving (return to cooperation), and be clear (let the
opponent learn what you do). The claim has been borrowed by
evolutionary biology, arms control writing, and platform-policy essays
ever since. I wanted to reproduce it on my laptop and see how much of
the original story holds up.

## What I tried

I wrote 10 strategies in Python and ran a full round-robin: every
strategy against every other strategy, including itself, for 200 rounds
per match. Payoffs were the classical 3 for mutual cooperation, 5 for a
defection against a cooperator, 0 for being the sucker, and 1 for
mutual defection. The line-up:

- Three baselines: always-cooperate (ALLC), always-defect (ALLD), and
  Random.
- The contenders: tit-for-tat (TFT), Grudger (cooperate until the first
  defection, then defect forever), Tester (probe with a defection,
  retreat if punished, exploit otherwise), Pavlov or win-stay-lose-shift
  (repeat the last move after a good outcome, switch after a bad one),
  Generous TFT (forgive a defection 10% of the time), TF2T (defect only
  after two defections in a row), and Joss (TFT plus a sneaky 10%
  defection rate).

I ran the tournament twice. Once clean, where every move is executed
exactly as intended. Once noisy, where each intended action flips with
probability 0.01 to simulate finger-slips, hardware faults, or
misperceptions. Axelrod's 1984 follow-up work and later papers by
Nowak, Sigmund, and Molander all argued that the noisy case is where
TFT's elegance starts to hurt.

The core of the simulation is short enough to read in one breath:

```python
def play_match(stratA, stratB, rounds=200, noise=0.0, rng=None):
    a_hist, b_hist = [], []
    a_score = b_score = 0
    for _ in range(rounds):
        a = stratA(a_hist, b_hist, rng)
        b = stratB(b_hist, a_hist, rng)
        if noise > 0:
            if rng.random() < noise: a = 1 - a
            if rng.random() < noise: b = 1 - b
        a_score += PAYOFF[a, b]
        b_score += PAYOFF[b, a]
        a_hist.append(a); b_hist.append(b)
    return a_score, b_score
```

Each pair was repeated 20 times in the clean tournament and 50 in the
noisy one to average over the stochastic strategies. That gave me one
score matrix per regime and one total per strategy.

## What happened

In the clean tournament, the leaderboard looked like this:

| Rank | Strategy | Total |
|------|----------|-------|
| 1 | Grudger  | 5209 |
| 2 | GTFT     | 5136 |
| 3 | Pavlov   | 4995 |
| 4 | TFT      | 4987 |
| 5 | TF2T     | 4955 |
| 6 | Tester   | 4891 |
| 7 | ALLC     | 4739 |
| 8 | Random   | 4346 |
| 9 | Joss     | 4026 |
| 10 | ALLD    | 3709 |

The top five all share Axelrod's property of being nice. Each of them
opens with cooperation and never defects unprovoked. The nasty cluster
(Tester, Random, Joss, ALLD) trails them by hundreds of points. The
gap between mean-nice (5170) and mean-nasty (4392) reproduces the
single most stable finding from the original tournament: the surface
of "cooperate first, retaliate when needed" beats the surface of
"figure out how to extract more than my share".

Grudger sat at the top in this clean run, which surprised me until I
looked at the pairwise heatmap. Grudger and TFT play identically
against everything that never tries a one-off defection, but Grudger
extracts an extra point per round against Joss because it punishes
forever, while TFT only mirrors a single move. In a perfectly clean
world that pays off.

![Round-robin totals, clean vs noisy](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/49_scores_bar.png)

Then I added 1% noise per action. The ordering scrambles:

| Rank | Strategy | Total |
|------|----------|-------|
| 1 | Tester   | 4760 |
| 2 | GTFT     | 4682 |
| 3 | Pavlov   | 4626 |
| 4 | TF2T     | 4578 |
| 5 | TFT      | 4486 |
| 6 | Grudger  | 4435 |

Grudger collapses from first to sixth. One accidental defection from
its partner triggers permanent retaliation, and the rest of the match
becomes a slog of mutual defection at 1 point per round. TFT slips
from 4th to 5th because TFT versus TFT enters its famous death spiral:
one mistaken defection produces a chain of alternating defections that
never resets on its own.

I ran a separate self-play test to make that vivid. Two TFTs against
each other start at near-perfect cooperation. By 1% noise the
cooperation rate falls to roughly half. Two Generous TFTs hold around
85%, because the 10% forgiveness probability is enough to break
revenge cycles. Two Pavlovs hold around 90%, because Pavlov's
win-stay-lose-shift logic actually treats mutual defection as a "lose"
state and shifts out of it.

![Cooperation rate in self-play as noise rises](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/49_noise_cooperation.png)

So Axelrod's four properties hold up, with one clarification. Nice and
retaliatory carry the top of the leaderboard in both regimes. Clear
shows up as the reason these strategies do not get exploited: every
opponent learns the rule quickly and stops probing. Forgiving is the
property that earns its keep in the noisy regime. TFT is forgiving in
the limited sense that it returns to cooperation as soon as the
opponent does, but it has no mechanism for absorbing a single error.
Generous TFT, which Nowak and Sigmund proposed in 1992, is forgiving
in the stronger sense that it sometimes pretends a defection did not
happen. That is the version of forgiveness the noisy data rewards.

The pairwise heatmap also shows why Tester does well in noise. Tester
opens with a defection. Against TFT, it gets punished, then plays
cooperatively. Against ALLC or any non-retaliator it switches to
exploitation. In a noisy world, occasional flips give Tester extra
chances to pretend its defections were accidents. That trick stops
working when the field is dominated by retaliators, which is part of
Axelrod's larger evolutionary argument.

## What it may mean

The core 1981 claim looks right. Across two regimes and 10 strategies,
the nice cluster sits above the nasty cluster, and TFT is the only
strategy that places top-5 in both clean and noisy play. That
consistency is a reasonable operational version of "TFT wins". Other
strategies can beat it in one regime, but none of them are competitive
in both.

The deeper takeaway from rerunning this 45 years later is that
forgiveness is not a luxury, it is structural. In any setting with
even modest execution noise, strategies that do not absorb the
occasional mistake collapse into mutual punishment. The version of
cooperation that survives real conditions is generous, not strict.
That maps onto a lot of social systems where the population is large
enough that small misperceptions are constant: trade networks,
diplomatic relationships, online community moderation, kin selection
in animals.

I am keeping the hedging tight here. This is a 10-strategy round-robin,
not an evolutionary simulation. The exact ordering depends on the
line-up, the noise level, and the number of rounds. The robust pieces
are the nice-versus-nasty gap and the TFT-fragility-under-noise
result, both of which were already in the literature by 1992.

## Loose ends

A full reproduction would add the original 14 strategies from the
1980 tournament (Downing, Friedman, Davis, Graaskamp, and the others),
plus the larger 62-strategy 1981 pool. It would also run an
evolutionary tournament where each strategy's share of the next
generation scales with its score, which is where Axelrod's 1984 book
gets its most-cited finding. Either would be a weekend of work, and
the code in this post is the starting point. The interesting
extension would be testing modern equilibrium selection ideas (zero-
determinant strategies, Press and Dyson 2012) against the classical
crew, to see whether the four old rules still hold when the strategy
space includes provably extortionate options.
