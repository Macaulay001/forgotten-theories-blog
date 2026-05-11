# Galton's 1894 bean machine still produces a clean bell curve at 200 rows

*Part 51 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1873 Francis Galton built a wooden box with a glass front, a hopper at
the top, a triangle of brass pins in the middle, and a row of slots at
the bottom. He called it the quincunx. He poured shot through it during
public lectures. The shot piled up into the same humped shape every time.
By 1889, in *Natural Inheritance*, he was using it as a teaching prop
for what he called "the supreme law of unreason". A redesigned two-stage
version followed in 1894.

The claim was simple and a little improbable. Each ball gets nudged left
or right at every pin, independently, with roughly equal probability.
After many rows the totals fall into a binomial distribution. As the
number of rows grows the binomial flattens into a Gaussian. Old idea,
older math (de Moivre had the limit in 1738), but Galton was the person
who made it physical. He wanted Victorian audiences to feel the central
limit theorem in their fingertips.

I wanted to see how cleanly the convergence happens at the scales Galton
could actually demonstrate, and what a slight bias does to the shape.

## What I tried

The simulation is short. Each ball is a row of Bernoulli draws. Sum the
draws and you have the bin it lands in. Run 10,000 balls per board.
Repeat at `n = 10` rows, `n = 50` rows, and `n = 200` rows. Then redo
each board with `p = 0.55` instead of `p = 0.5`, which is what you might
get from a slightly tilted machine or pins that lean one way.

For each board I compared the empirical histogram to the matching
Gaussian `N(np, np(1-p))`. The KS statistic on the standardised samples
(with a small uniform jitter to handle the discrete bins) gives a single
number for how far the curve sits from the limit.

```python
def run_board(n_rows, p, n_balls):
    steps = rng.random((n_balls, n_rows)) < p
    return steps.sum(axis=1)

for n in [10, 50, 200]:
    for p in [0.5, 0.55]:
        samples = run_board(n, p, 10_000)
        mu_th, sd_th = n * p, (n * p * (1 - p)) ** 0.5
        ks = kstest((samples + jitter - mu_th) / sd_th, "norm").statistic
        print(n, p, samples.mean(), samples.std(ddof=1), ks)
```

I set a single random seed so the numbers below are reproducible. The
whole thing runs in under two seconds on a laptop.

## What happened

At `n = 10` the histogram already looks like a bell. The empirical mean
sits at 5.001 against a theoretical 5.000, and the standard deviation is
1.582 against a theoretical 1.581. The KS statistic is 0.012, which
means the empirical CDF nowhere drifts more than 1.2 percent away from
the matching normal CDF on this sample.

![Galton boards: binomial samples vs Gaussian limit across n = 10, 50, 200 and p = 0.5, 0.55](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/51_galton_panels.png)

At `n = 50` the agreement gets a little better. Mean 25.021 against
25.000, sd 3.600 against 3.536, KS 0.010. At `n = 200` you get mean
100.028 against 100.000, sd 7.017 against 7.071, and KS 0.009. The KS
number does not collapse toward zero as fast as you might hope, because
we are bumping against Monte Carlo error from 10,000 balls. With more
balls it would keep falling.

| n   | p    | mean (emp / th)   | sd (emp / th)   | KS    |
|-----|------|-------------------|-----------------|-------|
| 10  | 0.50 | 5.001 / 5.000     | 1.582 / 1.581   | 0.012 |
| 50  | 0.50 | 25.021 / 25.000   | 3.600 / 3.536   | 0.010 |
| 200 | 0.50 | 100.028 / 100.000 | 7.017 / 7.071   | 0.009 |
| 10  | 0.55 | 5.512 / 5.500     | 1.579 / 1.573   | 0.016 |
| 50  | 0.55 | 27.504 / 27.500   | 3.518 / 3.518   | 0.006 |
| 200 | 0.55 | 110.010 / 110.000 | 7.009 / 7.036   | 0.008 |

The biased board behaves exactly as the binomial formula says it should.
A five percentage point tilt in the per-pin probability moves the mean by
ten bins on a 200-row board. The standard deviation barely shifts,
because `p(1-p)` is almost flat near `p = 0.5`. The shape is still
visually Gaussian; the whole curve has just slid to the right.

What I find slightly surprising is how good `n = 10` already looks. The
textbook line is that the binomial converges to the normal as `n` grows,
which makes it sound like you need many trials. Ten was enough on this
sample to keep the worst CDF deviation under two percent. Galton seems to
have known this. Photographs of his lectures show a quincunx with maybe
eight or nine rows, not fifty.

The one place a discrete-vs-continuous gap shows up is in the tails. At
`n = 10` the binomial has support `{0, 1, ..., 10}`, and the Gaussian
puts a small probability outside that range. The KS statistic captures
some of that. By `n = 200` the discreteness becomes invisible at the
plot resolution.

## What it may mean

If you accept the simulation at face value, Galton's demonstration is
about as honest as a 19th century teaching device gets. The setup makes
no smuggled assumptions. Each pin really does act like a Bernoulli draw,
the rows really are independent, and the limit really does kick in early
enough that a hand-built apparatus could show it to a Victorian audience.

The wider point may be that the central limit theorem feels almost too
generic. Many independent small shifts, none of them special, sum to a
Gaussian. The shape is not telling you something about the underlying
mechanism. It is telling you that the mechanism averages over enough
microscopic decisions for the details to wash out. That is also why
Gaussian fits to real-world data should make you slightly suspicious,
not relieved. A bell curve is the default outcome of summing noise, not
evidence that the noise has structure.

On a biased board the shift is mechanical. It does not require any
re-derivation. That matches what Galton was arguing in *Natural
Inheritance*: heritable traits that look Gaussian in a population can
still drift if the average parental contribution is tilted, without the
shape of the distribution changing. The board makes the algebra visible.

## Loose ends

I treated each ball as a perfectly independent sequence of Bernoulli
draws. A real quincunx has correlated deflections, jostling between
balls, and pins that wear unevenly. None of that is in the model. Adding
correlated steps would broaden the variance and might break the limit if
the correlations summed up.

The KS test on jittered samples is a workaround for comparing a discrete
distribution to a continuous one. A cleaner check would use the discrete
binomial PMF directly, with a chi-squared goodness-of-fit over the bins
that have enough mass. I expect similar conclusions but tighter numbers.

I also fixed `p` to be the same at every peg. Galton himself worried
about this in his 1894 redesign. He used a two-stage board where the
bottom half effectively resampled the top, which was his way of asking
whether the limit was sensitive to local irregularities. A pegwise
random `p ~ Beta(a, a)` would be the modern version of that question.
I would guess the limit still arrives at the same rate, but I have not
run it.

You can replicate the entire run in roughly two seconds on a laptop.
The code is short enough that any reader with NumPy installed can drop
it into a notebook and watch the bell curve assemble itself, which is
about as close as we can get to standing in front of Galton's lecture
hall in 1894.
