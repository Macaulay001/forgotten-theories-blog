# I ran Turing's 1952 reaction-diffusion equations on a 256x256 grid. The spots showed up.

*Part 33 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In the last paper he published before his death, Alan Turing made an
odd claim. Take two chemicals, an activator that makes more of itself
and an inhibitor that suppresses it. Let them diffuse on a flat sheet
at different rates. From a nearly uniform mixture, dotted with tiny
random fluctuations, spatial patterns will grow. Not waves that move
across the sheet and decay. Stationary patterns. Spots, stripes,
labyrinths. The kind of thing you see on a leopard, a tropical fish,
or a sand dune.

The paper sat awkwardly for about thirty years. Chemists could not
make a clean Turing pattern in a beaker until the CIMA reaction in
1990. Biologists, who had inherited a positional-information model
from Lewis Wolpert, were not keen on a rival mechanism. The 1995
Kondo and Asai paper on angelfish stripes was the first time most
working biologists took the math seriously. Now there is a small
industry of papers identifying activator-inhibitor pairs in
developing tissues, and the math even shows up in membrane chemistry.
The original claim, though, can be tested in an afternoon on a laptop.

## What I tried

I picked Gray-Scott, a particular reaction-diffusion system that John
Pearson made a clean phase diagram of in a 1993 *Science* paper. Two
species `u` and `v`, with the reaction `u + 2v -> 3v` and a feed-and-
kill term that keeps the system out of equilibrium:

```
du/dt = D_u ∇²u - u v² + F (1 - u)
dv/dt = D_v ∇²v + u v² - (F + k) v
```

`F` is the feed rate, `k` the kill rate. With `D_u = 0.16` and
`D_v = 0.08`, the inhibitor-side species diffuses half as fast as the
substrate it consumes, which is the wrong way around for the textbook
Turing intuition but works because of the cubic kinetics. The sweep is
across `F` from 0.014 to 0.078 and `k` from 0.045 to 0.065. Pearson's
diagram says you should see spots in the lower-right, stripes
diagonally across the middle, and a fairly large region where
everything decays to the trivial uniform state.

The numerical method is the simplest one that works. A 256x256
periodic grid, a five-point Laplacian stencil, forward Euler with
`dt = 1`. The initial condition is `u = 1`, `v = 0` everywhere, with a
small square of `u = 0.5, v = 0.25` in the middle, plus 2% Gaussian
noise. Each run is 8000 steps, which takes about ten seconds. Forty-
two parameter cells in total.

The whole simulation is about 12 lines of numerics. Here is the inner
loop, with the periodic Laplacian shown once:

```python
def laplacian(Z):
    return (np.roll(Z, 1, 0) + np.roll(Z, -1, 0)
            + np.roll(Z, 1, 1) + np.roll(Z, -1, 1) - 4 * Z)

for _ in range(steps):
    Lu, Lv = laplacian(u), laplacian(v)
    uvv = u * v * v
    u += dt * (Du * Lu - uvv + F * (1.0 - u))
    v += dt * (Dv * Lv + uvv - (F + k) * v)
```

That is it. No upwinding, no implicit step, no fancy boundary
treatment. Periodic edges, explicit time stepping, and the noise to
break symmetry.

## What happened

Patterns came out almost immediately, in the sense that by step 2000
you can see ring fronts radiating from the seed and by step 8000
those fronts have either stabilised into something interesting or
relaxed back to nothing.

Across the 42 cells I sorted by a small heuristic, using FFT peak
wavelength, the std of the v-field, and spot count from a connected-
component label:

| Regime | cells |
|---|---:|
| uniform / decayed | 25 |
| large-scale waves | 7 |
| spots | 5 |
| stripes or labyrinth | 4 |
| near-uniform low amplitude | 1 |

The spot cells live where Pearson said they would, near `F=0.025-0.030`
and `k=0.060-0.062`. One representative cell (F=0.030, k=0.062) ends
with 309 connected spots above a half-sigma threshold, a peak FFT
wavelength of 12.2 pixels, and a v-field standard deviation of 0.110.
The dots are roughly the same size. They do not move. If you re-run
with a different random seed, the dot *positions* change but the
distribution and size do not.

The stripe-and-labyrinth cells sit one row toward lower `k`. At
F=0.025, k=0.055 the field looks like a maze, with 50 components but
a clearly connected ridge structure that the component counter
underestimates. The peak wavelength there is also around 12 pixels,
which matches the linear-stability prediction that wavelength is set
by diffusion ratio and reaction rates, not by the feed and kill rates
directly.

The 25 "decayed" cells are the part of the phase diagram where the
fixed point at `(u, v) = (1, 0)` is linearly stable. Mostly that is
the high-`F`, high-`k` corner. The seed bump just diffuses away and
nothing replaces it. Visually those panels are uniformly black. They
matter, because they are evidence that the patterns are not a
numerical artifact: the same code on the same grid produces nothing
in regions where Turing's analysis says there should be nothing.

The phase-grid figure shows all 42 outcomes at once. The picture
matches Pearson's diagram qualitatively, though my classifier disagrees
with some of his boundary cells. The exemplar figure shows four cells
side by side: spots, labyrinth, large-scale waves, and the empty
decayed state.

One quirk worth noting. At F=0.014, k=0.05 I get a high-amplitude,
large-wavelength state with 147 components and a peak wavelength of
64 pixels, which my classifier labels "large-scale waves". Pearson
would probably call this `lambda` or `theta`, a slowly evolving
self-replicating pattern. With more steps it may settle, or it may
keep mutating. The line between "still pattern" and "ongoing
dynamics" is fuzzier than I expected, and I think this is one of the
real surprises of the experiment. The static-pattern half of Turing's
prediction is not always crisp. Some parameter cells live on the
boundary between stationary and slowly drifting.

## What it may mean

The qualitative claim from 1952 looks correct on this system, with
one ratio fixed and a single random seed per cell. Two locally
reacting species with different diffusion rates can produce
stationary spatial structure from noise, and the structure has a
selected wavelength that depends on rate parameters rather than
initial geometry.

This does not by itself say that real embryos use Turing systems.
That is a separate empirical question, and the answer in 2026 looks
like "sometimes, partly, for some tissues". Sheth et al.'s 2012 paper
on mouse digit patterning is one of the cleaner gene-knockout
demonstrations. Müller and colleagues' work on zebrafish has been
read both ways. The math does not pin down the chemistry.

What I take from running this myself is that the mechanism is
parameter-cheap and forgiving. You do not need to engineer the
initial condition. A patch of noise is enough. The system carries the
information about the eventual pattern in its rates, not its
boundary. That feels like the kind of mechanism evolution would find
again and again, and the convergent appearance of similar coat
patterns across mammals, fish, and even slime moulds may be a hint in
that direction. I would not press the analogy further than that on
the basis of one weekend's simulation.

## Loose ends

A single random seed per cell is not enough to be confident about the
boundary regions. With another week I would run 50 seeds per cell,
characterise the distribution of regime outcomes, and look for the
parameter cells where the system is genuinely bistable. I would also
sweep the diffusion ratio, since 2:1 is the conservative choice and
biological systems often have ratios closer to 10:1 or 100:1, which
would make patterns easier to find and the wavelength selection
sharper. The simulation code is short enough that anyone can
reproduce this in a few minutes on a laptop.
