# I simulated Blondlot's 1903 N-ray experiment with random noise. Confirmation bias alone reproduces the published effect.

*Part 104 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In late 1903 René Blondlot, professor of physics at the University of
Nancy and a respected member of the French Academy of Sciences,
announced a new form of radiation. He called it N-rays, for Nancy.
N-rays were emitted, he reported, by hot iron, by the sun, by tempered
steel under stress, and eventually by the human nervous system itself.
They were detected by watching a tiny gas spark in a darkened room and
judging whether it looked a little brighter when the source was on.
Within about 18 months roughly 120 papers from French laboratories
described N-ray spectra, polarisation, refractive indices, biological
emission. Then in September 1904 the American physicist Robert W. Wood
visited Nancy, quietly pocketed the aluminium prism that supposedly
dispersed the N-ray spectrum, and Blondlot's assistant continued to
read off spectrum positions in the dark just as before. Wood wrote a
short note in *Nature*. The literature stopped almost overnight.

I wanted to know whether the published N-ray effect could be
reproduced by an honest observer with no signal at all, just a
brightness threshold and a small dose of expectation. The textbook
answer is "yes, that's the whole point", but I had never seen anyone
put a number on it.

## What I tried

The N-ray detector was a human eye. That makes it natural to model as
a signal-detection problem. Let perceived brightness on a trial be a
random variable

    x ~ Normal(mean, sigma)

and let the observer report "I see it" when x exceeds some criterion
c. With no true signal the mean is zero. But Blondlot and his
collaborators worked under strong prior expectation: they believed the
source was on, they had a hypothesis to confirm, and they were
straining at the edge of vision. I added a single parameter beta that
shifts the perceived mean upward whenever the observer believes the
source is active. Everything else stays standard: sigma = 1, c = 1,
Gaussian noise, no memory across trials.

That gives three things to compute. First, the false-positive rate on
zero-signal trials as a function of beta. Second, the effect of
Wood-style blinding, which I modelled by simply zeroing beta (the
observer no longer knows whether the prism is in place, so belief no
longer shifts the mean). Third, the expected number of "discoveries"
in a 100-trial run at biases plausible for a motivated researcher in
a dim room.

I also pulled together a rough year-by-year count of N-ray papers
from the standard history-of-science reviews. The numbers are
approximate, but the shape is unambiguous: a sharp peak in 1903-1904,
then almost nothing.

## What happened

The simulation runs in a few lines.

```python
def trial(n, beta, sigma=1.0, c=1.0, believed_on=True):
    mean = beta if believed_on else 0.0
    x = rng.normal(mean, sigma, size=n)
    return (x > c).astype(int)

unblinded = trial(10000, 1.5, 1.0, 1.0, True).mean()
blinded   = trial(10000, 0.0, 1.0, 1.0, True).mean()
print(unblinded, blinded)
```

The unblinded condition (bias beta = 1.5, about one and a half noise
standard deviations of expectation) reports a positive on 69.0% of
zero-signal trials. The matched blinded condition reports 16.2%,
which is just the criterion-only baseline Phi(-1) you get for any
Gaussian noise crossing a one-sigma threshold. The gap is 52.8
percentage points, entirely from belief.

The fuller bias sweep:

| bias beta/sigma | positives per 100 trials |
|---|---|
| 0.0 | 15.9 |
| 0.5 | 30.9 |
| 1.0 | 50.0 |
| 1.5 | 69.1 |
| 2.0 | 84.1 |

A bias of one full sigma already gets you a coin-flip "detection"
rate. Two sigma gets you 84 positives out of 100 trials. For
comparison, the kind of effect Blondlot reported (a small but reliable
brightening when the source was on, a small but reliable dimming when
something blocked it) corresponds, in this model, to beta of order 1
to 2. So the published N-ray rate is recoverable from pure noise plus
modest expectation. No exotic physics, no fraud, no fakery: a
criterion shift inside the visual system is sufficient.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/104_bias_curve.png)

The historical literature curve is the other half of the story.
Counts from secondary reviews give roughly 40 papers in 1903, 80 in
1904, then 8, 2, 1, 0 in the following years. The drop tracks Wood's
*Nature* note in September 1904 with embarrassing precision. There
is no other case I know of in modern physics where a published effect
collapsed from a peak of 80 papers per year to zero inside 18 months
because one visitor removed a prism.

What surprised me was how cheap the bias has to be. I had imagined,
vaguely, that an experimenter-expectancy effect of this size would
require something like a confabulating observer. It does not. A
shift of one sigma in perceived brightness, applied silently by a
trained physicist who already believes the rays exist, is enough.
Single-blind protocols zero the shift; double-blind protocols zero it
and also remove the experimenter's ability to cue the observer.
Wood's prism trick was, in effect, an unannounced single-blind. The
rate did not change because the assistant's belief had not changed.

## What it may mean

The N-ray story is sometimes told as a French national embarrassment
or as a morality tale about scientific honesty. On this simulation it
is neither. Blondlot was almost certainly not lying. He was running
an unblinded judgment task with a near-threshold stimulus, and the
math of signal detection says you will see what you expect at rates
that look like real data. The only honest fix is to randomise the
on/off state outside the observer's awareness.

The same arithmetic may explain a non-trivial slice of single-lab
anomaly claims in modern physics and biology where the detector is
human judgment and the protocol is unblinded. Parapsychology
meta-analyses, animal-behavior coding without masked videos, expert
radiology second-reads on unmasked cases, certain chemistry
"observations" of new phases, any field where the measurement is
"does this trace look different to a trained eye". In all of those
the expected false-discovery rate at beta/sigma = 1 is around 50%.
That is enough to fill a literature.

I take three things away. The first is that experimenter-expectancy is
not a small correction; it is the dominant term when the signal is
near threshold. The second is that Wood's intervention was, methodo-
logically, ahead of its time: he did the first blinded replication
in modern physics, in a hotel room, with a pocketed prism. The third
is that the standard textbook treatment of N-rays is right but
under-quantified. The effect size needed to explain the entire 1903
literature is small enough that I would have missed it if I had been
in Blondlot's chair.

## Loose ends

The bias model is the simplest possible: Gaussian noise, constant
mean shift, fixed criterion. Real visual psychophysics is closer to a
log-Weber law and includes adaptation across trials. The historical
paper counts are approximate, drawn from secondary reviews rather
than a fresh database scrape. With another week I would fit beta to
actual modern psychophysics data on near-threshold spark detection,
add criterion drift across a session, and run the same model on the
polywater literature of the late 1960s, which I suspect has a very
similar shape. If anyone has photometric records from a 1903-era
spark gap, I would like to see them.
