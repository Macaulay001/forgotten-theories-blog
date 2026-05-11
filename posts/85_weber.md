# Fechner's 1860 logarithm loses to Stevens' power law on weight, brightness, and loudness by 35-53 AIC units

*Part 85 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Ernst Weber, working in Leipzig in the 1830s, ran a simple
experiment. He placed a small weight in a subject's hand and asked
how much extra had to be added before the subject noticed the
change. Then he doubled the baseline weight and asked again. The
just-noticeable-difference scaled with the baseline. Across many
trials he reported a relation that we now call Weber's law,

    dI / I = k

with k a small modality-specific constant (about 0.02 for lifted
weights, in his numbers).

Gustav Fechner, twenty-six years later, did the obvious calculus.
If every JND counts as one perceptual unit, and JNDs are a constant
fraction of the stimulus, then perceived sensation S as a function
of physical intensity I is the integral of dI/I, which is a
logarithm. He wrote the Weber-Fechner law,

    S = k * log(I / I0)

and built the whole field of psychophysics around it in his 1860
*Elemente der Psychophysik*. The law sat in textbooks for almost
a century.

Then in 1957 Stanley Smith Stevens, at Harvard, did something
Weber and Fechner never did: he asked subjects to assign a number
directly to a sensation. Not "is it different from the reference?"
but "this loudness is what fraction of that loudness?". The
magnitude-estimation curves Stevens collected did not look
logarithmic. They looked like power functions, S = k * I^a, with
exponents that depended on the modality. Brightness came out near
0.33, loudness near 0.60, and lifted weight near 1.45. Stevens
argued that Fechner had been wrong all along, and that perception
follows a power law, not a logarithm.

I wanted to look at this on actual numbers. Not to settle the
dispute (six decades of careful experiments have not settled it),
but to see what the disagreement looks like when you fit both
models to the same magnitude-estimation data with the same method.

## What I tried

I hand-encoded three small tables of magnitude-estimation data
from the standard psychophysics textbook range. The numbers are
read off published curves in Gescheider's *Psychophysics: The
Fundamentals* and Stevens' 1957 paper itself. They are
illustrative, not raw subject-level responses, but they preserve
the shape of the relationship that the original experiments
produced.

The three modalities:

- **Lifted weight**, seven points from 50 g to 3200 g. Stevens'
  exponent: ~1.45 (perceived weight grows *faster* than physical
  weight, the supra-linear case).
- **Brightness**, eight points spanning three log decades of
  luminance. Stevens' exponent: ~0.33 (compressive, the most
  famous case).
- **Loudness**, eight points spanning seven doublings of sound
  pressure. Stevens' exponent: ~0.60 (also compressive, the
  basis of the sone scale).

For each modality I fit two models. Fechner's gets the linear
form S = a + b * log(I). Stevens' gets the log-log form
log S = log c + d * log I, which is the standard way to fit a
power law and is what Stevens used himself. Then I scored each
fit with an AIC computed from the residual sum of squares on the
linear scale.

The whole thing runs in under a second. I did not pretend the
data were independent measurements with proper error bars. They
are textbook means. The AIC comparison is therefore suggestive,
not definitive, but the gap between the two models turned out to
be wide enough that the qualifier may not matter much.

## What happened

The power law won in every modality, by a wide margin.

![weber-fechner fits](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/85_weber_fechner_fits.png)

The recovered parameters:

| modality   | Stevens exponent d | AIC (Fechner log) | AIC (Stevens power) | delta AIC |
|------------|--------------------|-------------------|---------------------|-----------|
| weight     | 1.28               | 54.5              | 7.5                 | 47.0      |
| brightness | 0.31               | 6.3               | -29.3               | 35.6      |
| loudness   | 0.60               | 15.2              | -37.6               | 52.8      |

The convention is that a delta AIC above about 10 is a decisive
preference for the lower-AIC model. We get 35, 47, and 53. On
this data, Fechner's logarithm does not survive the comparison.

The recovered exponents are close to what Stevens reported.
Brightness came out at 0.31 against his 0.33. Loudness landed at
0.60, the textbook number to two decimals. Lifted weight came
out at 1.28, below Stevens' 1.45; the encoded points are spaced
too coarsely at the high end to anchor a steep supra-linear curve
with confidence, and the difference may be a fitting artefact
rather than a real disagreement.

The core of the calculation is short. The whole script is about
a hundred lines but the part that does the work is this:

```python
def fit_log(I, S):
    b, a = np.polyfit(np.log(I), S, 1)
    return a, b, S - (a + b * np.log(I))

def fit_power(I, S):
    d, logc = np.polyfit(np.log(I), np.log(S), 1)
    c = np.exp(logc)
    return c, d, S - c * I**d

def aic(resid, k):
    n = len(resid)
    return n * np.log(np.sum(resid**2) / n) + 2 * k
```

Eyeballing the figure helps. On a log-x scale, Fechner predicts a
straight line for sensation against intensity. Stevens predicts a
curve. The brightness and loudness panels show the data clearly
curving upward away from the dashed Fechner line at high
intensities. The weight panel shows the opposite curvature; even
though both models are wrong about something, the power law sits
closer to the data points across the whole range.

It is worth saying that Weber's *original* statement, dI/I = k,
is not what loses here. Weber's law is about the JND near
threshold, and several modern measurements confirm it as a rough
approximation in mid-intensity ranges. What loses is Fechner's
*integration* of Weber's law, which assumed that all JNDs are
perceptually equal. That auxiliary assumption is the part Stevens
challenged, and on these curves the power-law form fits better.

## What it may mean

If the world is logarithmic in perception, then doubling the
acoustic energy doubles the perceived loudness in the limit. If
the world is a power function with exponent 0.6, then doubling
the acoustic energy multiplies the perceived loudness by 2^0.6,
about 1.52. The two predictions diverge by about a third for
every doubling. Engineers building audio level meters, display
gamma curves, and clinical pain scales have to pick one, and the
working assumption since about 1960 has mostly been Stevens'.

The result here is a small reminder that integration can introduce
its own assumption. Weber's differential statement (sensitivity
scales with stimulus) is easy to motivate from neural adaptation:
a receptor whose firing rate is a saturating function of input
will have a JND that grows with baseline. But going from there to
"perceived magnitude is the integral of JNDs" requires that
perceptual distance is additive in JNDs, which is a separate
psychological claim. Stevens' magnitude-estimation paradigm
sidesteps that claim by asking subjects directly. Whether one
trusts the direct number-assignment is a different question, and
the literature on cognitive biases in magnitude estimation is
large.

This is not a refutation of Fechner. It is a fit comparison on
three coarse curves, in line with the well-known finding that
power laws describe magnitude-estimation data better. Where the
log form still appears useful is in summarising the JND data
itself, and in engineering scales like the decibel where the
relevant question is dynamic range rather than perceptual
distance.

## Loose ends

Three modalities is not many. The Stevens table has roughly
thirty entries (electric shock, taste of salt, smell, vibration,
visual length, temperature) and the exponents range from about
0.3 to over 3. A proper test would refit all thirty against the
Fechner log and tally how many invert the AIC sign; my guess is
that almost all of them would still favour the power law, but
with smaller margins for modalities where the data span less than
two decades of intensity.

The other thing worth doing is fitting at the subject level. Each
of the textbook curves is a group median, and group medians can
flatten curvature. With another week I would pull a recent
loudness magnitude-estimation dataset, fit per-subject power
exponents, and ask how the exponent distributes. Some of the
spread in Stevens' table likely reflects real between-subject
variation rather than purely between-modality differences. You
can replicate the figure here in under a minute on a laptop.
