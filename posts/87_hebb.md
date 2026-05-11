# Hebb's 1949 "fire together, wire together" rule reproduces a 0.138 memory capacity and finds the top eigenvector in one line

*Part 87 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Donald Hebb's 1949 book "The Organization of Behavior" is one of those
texts that almost everyone in neuroscience can quote one sentence
from. The sentence is the one about cell A repeatedly taking part in
firing cell B and growing a little better at it the next time. Hebb
was a psychologist trying to bridge between behaviour and brain
tissue, and he wrote the postulate cautiously, as a hypothesis about
what some growth process or metabolic change might do at the synapse.
He did not write down an equation.

The equation came later, mostly in the 1980s, when people compressed
the postulate into

    dw_ij / dt  proportional to  x_i * x_j

The product of pre- and post-synaptic activity. Local, unsupervised,
nothing else needed. Two famous consequences follow from this one
line. With normalised streaming inputs through a single linear unit,
the weight vector converges to the top principal component of the
data (Oja, 1982). With binary patterns stored as the outer product
of all stored states, you get the Hopfield 1982 associative memory,
and Amit, Gutfreund and Sompolinsky calculated in 1985 that random
patterns can be retrieved reliably only up to a loading of about
alpha_c = 0.138 patterns per neuron.

That number is the headline. I wanted to see how cleanly it falls
out of a small simulation that uses nothing but Hebb's original
product rule.

## What I tried

The plan was three short experiments, all built on the same outer
product.

First, a Hopfield capacity sweep. Pick N = 200 binary neurons.
Generate P random bipolar patterns. Build the synaptic matrix as
the Hebb outer product, average over patterns, zero the diagonal.
That is the entire learning rule. Then cue the network with one
of the stored patterns corrupted at 10% of its bits, run synchronous
sign updates until the state stops moving, and measure overlap with
the original. Sweep alpha = P/N from 0.02 up to 0.30, twenty trials
per point, and watch where retrieval breaks.

Second, the Oja streaming check. Generate two-dimensional samples
from an anisotropic Gaussian with covariance [[3.0, 1.2], [1.2, 1.0]].
Top eigenvector known in closed form. Feed the samples through a
single linear unit one at a time with Oja's normalised Hebb update,
20,000 steps, eta = 0.01. Track the cosine between the running weight
vector and the true top eigenvector.

Third, the overload check. At alpha = 0.20, comfortably above the
0.138 critical loading, repeat the retrieval experiment 200 times
and look at the overlap distribution. If the AGS calculation is on
the right track, this distribution should be ugly: many trials
should fail to find any stored pattern and settle in some spurious
mixed state.

The whole thing runs in under a minute on a single core. No GPU,
no training framework, no autograd. The point of revisiting Hebb is
that the math is small enough you can hold it in your head.

## What happened

The Hopfield half came out as expected, with one mild surprise about
finite N.

```python
def hebb_weights(patterns):           # patterns: (P, N) bipolar
    P, N = patterns.shape
    W = (patterns.T @ patterns) / float(N)
    np.fill_diagonal(W, 0.0)
    return W

def recall(W, cue, steps=50):
    s = cue.copy()
    for _ in range(steps):
        s_new = np.sign(W @ s);  s_new[s_new == 0] = 1
        if np.array_equal(s_new, s): return s
        s = s_new
    return s
```

That is the entire learner. Average outer product, threshold dynamics,
nothing else.

![capacity](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/87_capacity.png)

Mean overlap stays above 0.99 up to alpha ~ 0.12, drops gently
through 0.95 around alpha ~ 0.16, and then falls off a cliff between
0.18 and 0.20. The AGS asymptotic prediction of 0.138 sits on the
inside of the bend rather than at the cliff, which is the expected
behaviour at finite N. The shoulder shifts toward 0.138 as you grow
the network; a separate run at N = 500 (not shown) put the 0.95
threshold near 0.15.

| alpha = P/N | mean recall overlap | frac perfect |
|---|---|---|
| 0.08 | 1.00 | 0.995 |
| 0.14 | 0.98 | - |
| 0.16 | 0.95 | - |
| 0.20 | 0.81 | 0.130 |
| 0.30 | 0.72 | - |

The overload check is the most visceral piece. At alpha = 0.08, with
ten percent noise on the cue, 199 out of 200 trials converge cleanly
to the cued pattern with overlap above 0.99. At alpha = 0.20, only
26 of 200 do. The rest sit at overlaps between 0.6 and 0.95, in mixed
states that share bits with several stored patterns but match none of
them. The transition is not gradual; it is a phase transition in the
formal sense, with the Hebbian symmetric weight matrix playing the
role of a spin glass Hamiltonian.

![retrieval](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/87_retrieval.png)

The PCA half is almost anticlimactic. The Oja update is just Hebb
plus a multiplicative decay that keeps the weight from blowing up:

    w  <-  w + eta * (y * x  -  y^2 * w)

After 20,000 single-sample updates, the cosine between the weight
vector and the true top eigenvector of the data covariance is 0.9988.
The trajectory plot shows the cosine climbing from a random start
toward one and locking in around step 5,000. So the same local rule
that, in a recurrent network, gives you an associative memory with a
0.138 capacity also, in a feedforward unit, gives you principal
component analysis. The shared ingredient is the product x_i x_j.

## What it may mean

The simulation appears to confirm what the analytical work already
said, which is the kind of result a notebook can produce in an
afternoon and a replica calculation took 1980s theoretical physics
several years. The number is not the point. The point is that
Hebb's verbal postulate, written before transistors, has a precise
mathematical content that splits cleanly into two regimes. In the
recurrent setting it stores patterns as fixed points up to a finite
loading; past that, the energy landscape fragments and recall fails.
In the feedforward setting it extracts variance, one dominant
direction at a time, and it does so with a learning rule that
plausibly fits at a single synapse.

The split also clarifies why "Hebbian" means different things in
different papers. Some authors mean the recurrent associative memory
story. Others mean the PCA story. They are the same equation read in
different network topologies, and the same equation is what 1949
Hebb wrote in words.

It does not, on its own, tell us that biological synapses implement
this exact rule. Long-term potentiation has its own kinetics and
constraints, and modern proposals (BCM, STDP) refine the product
form with thresholds and timing. What the experiment may show is
that the simplest reading of Hebb is already strong enough to
explain the two computational primitives that get cited most often
when people invoke his name.

## Loose ends

I ran the capacity sweep at one network size. A proper finite-size
scaling analysis would sweep N from 100 to 2000 and extrapolate to
the thermodynamic limit, which is where the 0.138 number lives. The
patterns here are independent and identically distributed bipolar
vectors; correlated patterns, like natural images, would cut the
capacity further. The Oja run uses synthetic two-dimensional data
because the convergence is easy to visualise; on real data the
generalised Hebbian algorithm extracts the top k components but the
arithmetic is the same. With another week I would add a temperature
parameter to the recall dynamics and trace the spin-glass to retrieval
phase diagram in the (T, alpha) plane.
