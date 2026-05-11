# I scored 175 arXiv abstracts for Popper's "risky prediction" rule. String theory came in at 23%.

*Part 30 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1934 Karl Popper published a small book in Vienna with a big claim. A
theory is scientific, he said, only if it makes risky predictions that
could turn out wrong. Einstein's 1919 eclipse forecast was his hero
example. Marxism and psychoanalysis were his villains, because their
adherents always seemed to find a way to absorb whatever happened. The
rule is famous, repeated in every philosophy-of-science 101, and also
kind of awkward, because nobody ever counts.

I wanted to count. Specifically I wanted to see whether one of the
older critiques of modern theoretical physics, that string theory and
its cousins make few risky predictions, would show up if you actually
scored a stack of recent abstracts. The critique has been made
loudly by Lee Smolin, Peter Woit, Sabine Hossenfelder, and quietly by
quite a few working physicists. It has also been pushed back on, fairly,
by people who point out that theory takes time.

So a simple test. Pull recent abstracts from three arXiv categories
that correspond to "theory", "experiment", and "observation" in
high-energy physics. Score each on two binary questions: does it state
a falsifiable prediction, and does it report a result that could have
come out negative? Then look at the rates.

## What I tried

The three categories were `hep-th` (high-energy theory, where string
theory and most of its formal cousins live), `hep-ex` (high-energy
experiments, mostly LHC and B-physics), and `astro-ph.HE` (high-energy
astrophysics, which is observational). I pulled the most recent 60
abstracts from each through arXiv's Atom API. The API rate-limited me
twice, but with long backoff I got 175 unique papers in the end.

Then I asked GPT-4o-mini, through OpenRouter, to read each abstract and
return strict JSON with two booleans. The prompt instructed it to call
a paper "falsifiable" only when there was a specific quantitative claim
that could be checked against future data, and "risky-result" only
when the abstract reported a measurement or test whose outcome could
have come out the other way. I also asked for a short reason so I
could spot-check by hand. Total LLM cost: $0.016 for the run, well
under the $0.10 cap I had set.

The classification is not gospel. It is one cheap LLM's reading of
175 abstracts. I read eight of them myself and the model agreed on
all eight, which is encouraging but not a real audit. A real audit
would use a second model and human adjudication for disagreements.

```python
# core of the prompt sent for each abstract
prompt = f"""Title: {title}
Category: {cat}
Abstract: {abstract}

1. falsifiable_prediction: does this state a specific quantitative
   prediction that could be checked against future data?
2. risky_result_reported: does it report a measurement or test whose
   outcome could have come out negative?

Return {{"falsifiable_prediction": bool, "risky_result_reported": bool,
         "reason": "<=20 words"}}"""
```

Aggregating across categories, with Wilson 95% intervals because the
sample sizes are modest.

## What happened

| Category    | n  | Falsifiable prediction | Risky result reported |
|-------------|----|------------------------|-----------------------|
| hep-th      | 60 | 0.45 (0.33-0.58)       | 0.23 (0.14-0.35)      |
| hep-ex      | 59 | 0.83 (0.72-0.91)       | 0.73 (0.60-0.83)      |
| astro-ph.HE | 56 | 0.89 (0.79-0.95)       | 0.80 (0.68-0.89)      |

The gap is large and the confidence intervals do not overlap. A
two-proportion z-test on hep-th vs hep-ex for the risky-result column
gives z = 5.6, p < 1e-7 on this sample.

The texture of the abstracts under each label is what you would expect.
A typical hep-th paper that got flagged as neither falsifiable nor
risky was something like "Hamiltonian formulation of the
supersymmetric KdV equation". The classifier wrote: "focuses on
formalism without specific predictions or experimental results". A
hep-ex paper at the other end: "Evidence for the decay B0s -> phi
eta', branching ratio...". Falsifiable and risky. An astro-ph.HE one:
"Young Massive Star Clusters as TeV Emitters: Constraints from H.E.S.S.
and LHAASO". The Popperian shape is right on the surface.

I want to flag the hep-th 45% on the first column, because that is
higher than the Smolin-style critique would suggest. Quite a lot of
theoretical work does state quantitative claims, things like "in this
limit the entropy scales as", or "the partition function on this
manifold equals". These are checkable in principle, even if no
experiment exists. The drop to 23% on the second column is where the
real gap sits. Theory papers state things that could be wrong, but
they rarely report a moment where something actually got tested and
could have failed.

Experimental and observational papers, by contrast, are essentially
nothing but those moments. If you remove the experimental papers that
do not report a measurement, you are left mostly with detector design
documents and review articles. The base rate is pinned high by the
genre.

A small caveat that matters. The classifier sometimes called a
derived signature "falsifiable" without the signature being measurable
in any near-term experiment. "This model predicts a peak in the
gravitational wave spectrum at 10^-18 Hz" is, strictly, falsifiable.
But it is not falsifiable now, or this decade, or by any planned
instrument. I did not try to filter for "feasibly falsifiable" because
I did not want to bake my own judgment into the score. The honest
reading is that the 45% number for hep-th is an upper bound on
"actually checkable" predictions in the field.

The other thing worth saying: hep-th is not just string theory. It is
also lattice field theory, conformal field theory, integrable systems,
holography, and quantum information adjacent work. Some of those
subfields do connect to experiment, and the Popper rates inside hep-th
are presumably not uniform. A finer breakdown would be worth doing
with a larger sample.

## What it may mean

The Popper rule, treated as a measurement, gives a clean number on
modern arXiv. On this sample, theoretical high-energy physics looks
roughly half as Popperian as its experimental and observational
neighbours, and the gap is widest on the "risky result actually
reported" axis. That is consistent with the long-standing complaint
that the theoretical end of the field has drifted into a regime where
its outputs are checked mostly against internal consistency, not
against the world.

This is not a verdict on string theory. Popper's criterion was always
crude, and you can write deep, useful physics that does not pass it.
Mathematics often does not pass it either. What the numbers suggest is
that a working physicist's intuition (that hep-th papers feel
different from hep-ex papers) has a measurable shape.

If anyone wanted to make this real, the next step would be a
cross-decade scan. Take 500 hep-th abstracts per year from 1994 to
2024, run the same classifier, and see whether the risky-result rate
in theory has fallen, stayed flat, or recovered. That would cost
maybe $2 in LLM calls and an afternoon to wire up.

## Loose ends

A second classifier (Claude Haiku, or Llama 3.3 70B) should re-run
the same labels and the disagreements should be read by hand. The
sample is recent only; an older slice from 1995 would test whether
the gap is new or steady. Finer arXiv subcategories within hep-th
would tell us whether the low Popper score is concentrated in
string-phenomenology or spread evenly. And someone with a stronger
philosophy background should sharpen the prompt; "risky" is doing
real work in Popper, and one cheap LLM is a blunt judge of it.
