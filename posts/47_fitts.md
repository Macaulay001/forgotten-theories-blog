# I rebuilt Paul Fitts's 1954 reaching task in numpy. His log-bit law fits to four decimal places.

*Part 47 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1954 Paul Fitts, an Air Force psychologist at Wright-Patterson, sat
subjects in front of two metal plates and had them tap back and forth
between them with a stylus. He varied the distance between plates and
the width of the plates and measured how long each tap took. Out of
that came one of the cleanest empirical rules in psychology: the time
to point at a target grows linearly with the log of the ratio of
distance to width. He wrote it as MT = a + b·log2(2D/W) and called the
quantity inside the log the index of difficulty, in bits, by direct
analogy with Shannon's channel capacity formula published six years
earlier.

The reason this matters in 2026 is that you have used it today. Every
button you tapped on a phone, every dropdown you grabbed in a browser,
every joystick flick in a game, is sized inside a Fitts envelope.
Apple's Human Interface Guidelines and the W3C accessibility guidance
both bake the law in. It quietly survived behaviorism, the cognitive
revolution, and the touchscreen. I wanted to see how cleanly it
actually fits when you generate data from a plausible motor noise
model and turn the crank.

## What I tried

The plan was small. Pick five movement distances (5, 10, 20, 40, 80 cm)
and five target widths (5, 10, 20, 40, 80 mm). That gives 25 (D, W)
combinations. Drop pairs with index of difficulty outside roughly
[1, 7] bits, since that is the range Fitts and his successors stayed
inside. Simulate 40 reaches per condition. Generate each movement time
from MT = 0.150 + 0.150·ID + epsilon, where epsilon is Gaussian with
35 ms standard deviation. Those constants are roughly what the
literature reports for index-finger tapping: a 150 ms reaction-time
floor and 150 ms per added bit of difficulty, which works out to about
6.7 bits per second of motor throughput.

Then fit the data four ways.

The first three are competing log formulations that all claim to be
the right one. Fitts's original is MT = a + b·log2(2D/W). Welford in
1968 argued for log2(D/W + 0.5) because Fitts's form goes negative for
tiny ID. MacKenzie in 1989 preferred log2(D/W + 1) on information-
theoretic grounds, claiming it better matches Shannon's actual capacity
expression. All three converge at high ID and disagree mainly when D
and W are close.

The fourth fit is the serious historical rival. Crossman in 1957
proposed a power-law alternative, MT = a + b·(D/W)^k, on the grounds
that motor control is at root biomechanical rather than informational,
and that the log shape is a small-range approximation. The power law
was treated as a credible competitor through the 1960s before slowly
losing ground.

## What happened

The original Fitts form recovered its own generative constants almost
exactly. Estimated intercept 147 ms against a true 150 ms, slope 150
ms per bit against a true 150, R² of 0.983. The Welford and MacKenzie
variants land within 0.005 of that R², which matches the
indistinguishability that MacKenzie himself reported in 1992 when he
refit dozens of published datasets.

Here is the core of the fit, with all the imports stripped:

```python
ID_fitts = np.log2(2 * D / W)
slope, inter = np.polyfit(ID_fitts, MT, 1)
yhat = inter + slope * ID_fitts
r2 = 1 - np.sum((MT - yhat)**2) / np.sum((MT - MT.mean())**2)
# slope = 0.1502 s/bit, inter = 0.1474 s, r2 = 0.9832
```

The table of formulations:

| Formulation | Intercept (s) | Slope (s/bit) | R² |
|---|---|---|---|
| Fitts log2(2D/W) | 0.147 | 0.150 | 0.9832 |
| Welford log2(D/W+0.5) | 0.241 | 0.161 | 0.9819 |
| MacKenzie log2(D/W+1) | 0.190 | 0.171 | 0.9788 |
| Crossman a + b(D/W)^k | -62.4 | 62.7 | 0.9832 |

The Crossman row is the part I did not expect. The optimizer fit a
power law to the same data and reported the same R² as Fitts. Look at
the constants. Intercept of negative 62 seconds. Slope of 62.7. And
the exponent k came back at 0.003.

That is the optimizer finding a numerical loophole. When k is very
close to zero, (D/W)^k expands as 1 + k·ln(D/W) + O(k²). Plug that into
a + b·(D/W)^k and you get a + b + b·k·ln(D/W), which is just a log
function with two large constants (a and b) that cancel each other
out. The power law fit only by becoming a log fit in disguise, with
nonsense values for the named parameters. That is a failure mode, not
a success. On data this log-shaped, the power model has no honest
solution.

![Fitts law linear fit and residuals across formulations](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/47_fitts_linear_fit.png)

The right panel of the figure shows residuals for the three log
formulations stacked on the same axes. They all sit inside a ±80 ms
band with no obvious trend against ID, which is what you expect when
the residuals are pure Gaussian motor noise. There is no curvature
left over for a power law to claim.

A throughput estimate falls out of the slope. 1 divided by 0.150 s per
bit gives 6.67 bits per second of effective motor bandwidth, which
sits in the 4 to 10 bits per second band reported for index-finger
pointing in the modern literature (Soukoreff and MacKenzie, 2004). The
generative model knew nothing about throughput. The number came back
correct because the model is just Fitts's law plus noise, and the law
is internally consistent at the parameter scale Fitts chose.

## What it may mean

The most interesting thing here may be how thoroughly the log form
beats its main rival on its home turf. Crossman's power law is not
wrong, exactly, but it appears to need either very low-ID ballistic
movements or specific neuro-mechanical regimes to look better than the
log. On clean Fitts-shaped data the power model degenerates into a
disguised log.

That has a quiet implication for UI design. The reason 44-pixel touch
targets work is not that someone empirically tuned the number against
a focus group. It is that the motor system seems to operate in a
log-bit regime, where doubling the target width gives a constant time
saving rather than a proportional one. If the rule were a power law
the math behind accessibility guidance would look different, and so
would the cost of small buttons.

The other open thread is whether modern motor learning models, the
kind trained from raw EMG or video, rediscover the log scaling
without being told to. If they do, Fitts's information-theoretic
framing seems to have caught something real about how nervous systems
move limbs. If they do not, the rule may be an emergent feature of
specific tasks rather than a deep statement about motor capacity.

## Loose ends

The data is synthetic. Real subjects show floor effects below 200 ms,
curvature near ID = 0, and amplitude-scaled noise that this model
ignored. The speed-accuracy tradeoff, the other half of Fitts's
original paper, did not appear here at all because I did not simulate
endpoint errors. With another week I would replay the analysis on the
public Soukoreff-MacKenzie reanalysis dataset and check whether the
Crossman power law degenerates the same way on real reaches, or
whether actual human noise leaves enough curvature for a power law to
do honest work.
