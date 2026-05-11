# Galton's 1907 ox crowd nailed the weight to 0.75 percent. I checked when that trick stops working.

*Part 23 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1906, Francis Galton went to a country fair in Plymouth and watched
787 people write down their guess for the dressed weight of an ox. The
prize for the closest guess was a small one. Galton bought the cards,
took them home, and worked out the statistics. He published the result
in *Nature* in March 1907 under the title "Vox Populi".

The crowd's median guess was 1207 pounds. The actual weight, once the
ox was butchered and weighed, was 1198 pounds. The crowd was off by 9
pounds, or about 0.75 percent. Galton liked that the median was so
close because he distrusted the mean, treating the median as the more
"democratic" estimator: each voter has one vote and outliers can't
shout louder than anyone else.

The story has been quoted in every popular book on collective
intelligence since Surowiecki wrote *The Wisdom of Crowds* in 2004. It
gets used to justify prediction markets, polling aggregators, and the
idea that averaging Twitter takes might tell you something. What I
wanted to know was simpler. When does the trick stop working?

## What I tried

The 1906 cards have actually been digitised (Wallis 2014, *Significance*),
but for this short test I used Galton's own reported quartiles and ran
a simulation that lets me vary three things he could not.

First, the noise shape. Galton's crowd happened to be roughly symmetric
around the truth. If everyone systematically lowballs or overshoots, as
people often do for unfamiliar quantities, what happens? I drew guesses
from a lognormal distribution with adjustable bias and spread.

Second, crowd size. Galton had 787. I tested 1, 3, 10, 30, 100, 300,
787, and 3000 to see whether error really shrinks as 1/sqrt(N).

Third, and this is the one I cared about, correlation between guessers.
At a country fair in 1906 you wrote your number on a card and put it in
a box. You couldn't see what anyone else wrote. On a modern social
platform, everyone sees everyone, and a single confident post can
anchor thousands. I modelled this with a shared latent bias of strength
rho, mixed with independent individual noise. At rho=0 guessers are
fully independent, the Galton fair scenario. At rho=1 every guesser
gives the same answer.

For each setting I drew 2000 trials of crowds, recorded mean and median,
and compared to the true weight of 1198 pounds.

## What happened

Under symmetric Gaussian noise with sigma=150 pounds and N=787, the
crowd mean averaged 4.3 pounds of error and the median 5.3 pounds. A
single guesser was off by about 120 pounds. The crowd was roughly 25
times sharper than its typical member, which is in the ballpark of
Galton's reported 4.9x advantage given his tighter underlying spread.
Both mean and median tracked the 1/sqrt(N) curve cleanly.

The lognormal experiment was more interesting. When guesses are
right-skewed (a long tail of high overestimates) but unbiased in log
space, the mean chases the tail and lands at 24 pounds of error; the
median sits at 8. That is the regime where Galton's preference for the
median actually buys you something. Once a systematic bias creeps in,
say a log-bias of 0.10 meaning the typical guesser overshoots by about
10 percent, both aggregators inherit it. Mean error rises to 152
pounds, median to 126. The crowd is still better than an individual
(238 pounds typical error) but it has lost most of its precision.

The correlation result was the cleanest finding.

```python
# shared bias of strength rho, individual noise of strength 1-rho
z_shared = rng.normal(0, sigma, size=(trials, 1))
z_indiv  = rng.normal(0, sigma, size=(trials, n_guessers))
guesses  = truth + np.sqrt(rho) * z_shared + np.sqrt(1 - rho) * z_indiv
crowd_err  = np.abs(guesses.mean(axis=1) - truth).mean()
indiv_err  = np.abs(guesses - truth).mean()
```

At rho=0 the crowd of 787 averaged 4.3 pounds of error. At rho=0.05
that jumped to 28. At rho=0.20, fifty-three pounds. At rho=0.80, the
crowd of 787 was only 11 percent better than a single guesser. The
"wisdom" eroded fast.

| rho  | crowd err (lb) | crowd / individual |
|------|----------------|--------------------|
| 0.00 | 4              | 0.04               |
| 0.05 | 28             | 0.23               |
| 0.20 | 53             | 0.44               |
| 0.50 | 86             | 0.71               |
| 0.80 | 106            | 0.89               |

Intuitively the math is forced: the variance of the crowd mean under
shared bias does not go to zero with N, it goes to rho times the
individual variance. You can throw a million guessers at the problem
and they will not beat the shared bias.

I also re-read Galton's own quartiles. He gave Q1 around 1170 and Q3
around 1244, an IQR of 74 pounds. That implies a per-guesser sigma of
about 55 pounds, so a typical guesser was off by about 44 pounds. The
crowd's 9-pound median error is roughly 4.9 times better than the
typical guesser. That number is consistent with low correlation, well
below the 0.2 threshold where things fall apart. The 1906 fair, in
other words, was close to the ideal case.

## What it may mean

The "wisdom of crowds" is real but conditional. The crowd at a Plymouth
country fair worked because cards were filled out in silence, because
butchers and farmers had genuine but imperfect priors on cattle weight,
and because no one had a megaphone. Three properties: independence,
informed guessing, and no public anchor. Remove any one and the
benefit shrinks fast.

In the simulation, a shared-bias correlation of 0.2 cuts the crowd
advantage in half. That feels like a low bar to clear in a modern
information environment. Polling aggregators may already sit above it
when pollsters share methodology biases. Prediction markets may sit
above it when traders read each other. I am not saying these are
useless. I am saying their accuracy floor is set by their correlation,
not by their size.

Galton's preference for the median (over the mean) was the right call
for skewed noise but slightly aesthetic in his own data. The mean would
have been even closer (he noted this, with mild reluctance, in his
follow-up letter).

One more thing worth pointing out. The advantage factor scales with
how informed individuals are. If a crowd has a sigma of 55 pounds (the
implied 1906 number), the crowd error at perfect independence is on
the order of 2 pounds for N=787. If individuals are wildly uninformed,
sigma=500 pounds say, the crowd error becomes 18 pounds, still better
than any single voter but no longer striking. Crowds compress noise,
they do not invent knowledge. That feels obvious when stated, but it
is missing from most retellings.

A second observation. People sometimes argue that averaging over
"experts and amateurs" is the magic. In my simulation, having a few
well-informed guessers (low sigma) mixed with many noisy ones (high
sigma) helps less than you would think, because the noisy guessers
dominate the variance. A weighted mean by inverse variance recovers
the experts' precision, but that requires knowing who the experts
are, which Galton did not.

## Loose ends

I did not pull the digitised 787 cards from the Wallis 2014
reconstruction. That would let me check the actual noise shape rather
than infer it from quartiles. I also did not test heavy-tailed
distributions where the population mean is undefined; in that regime
the median has to win, and it might be the more honest default. With
another week, the useful test would be to take a modern crowd guessing
dataset (Good Judgment Open forecasts, say) and estimate the empirical
rho across forecasters. If that number sits above 0.2 the popular
framing of the Galton story needs an asterisk.

You can replicate the simulation in about thirty seconds. The full
script is at `23_galton_crowds/src/simulate.py` in the repo.
