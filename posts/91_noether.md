# Noether's 1918 theorem holds to 14 decimals in a symplectic Kepler orbit

*Part 91 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In July 1918, Emmy Noether published a short paper in the Göttingen
*Nachrichten* with a title that sounds like an exercise: *Invariante
Variationsprobleme*. Inside was one of the cleanest results in
mathematical physics. If the action of a system is invariant under a
continuous transformation, there is a quantity that is exactly
conserved by the equations of motion. Time-translation gives energy.
Space-translation gives linear momentum. Rotation gives angular
momentum. The same machinery hands you electric charge from gauge
invariance.

Noether was 36, working without pay in a department that would not
let women hold faculty positions. Hilbert had pulled her to Göttingen
to sort out the conservation puzzles raised by general relativity,
and her theorem did that and much more. It became one of the load-
bearing beams of twentieth-century physics. The argument is short
enough that a careful afternoon at a notebook can re-derive it. I
wanted to feel it from the other side, by watching the conservation
laws appear and disappear under symmetry-breaking.

## What I tried

The plan was four small numerical experiments, each isolating one
piece of the theorem.

First, the Kepler two-body problem. A point mass in a 1/r potential
has time-translation symmetry (energy) and rotational symmetry
(angular momentum) but no spatial-translation symmetry (because the
origin is special). I picked an elliptical orbit with semi-major
axis a = 1 and eccentricity e = 0.6, then integrated it for 20 orbits
with a velocity-Verlet (Störmer-Verlet) step at dt = 1e-3. Verlet is
*symplectic*: it preserves the phase-space volume form exactly even
when it does not preserve E exactly. The empirical consequence is
that E oscillates around the true value rather than drifting.

Second, an undriven pendulum at large amplitude (initial angle
~1 rad, well outside the small-angle linear regime). Same leapfrog
scheme. Time-symmetry intact, so energy should hold.

Third, a driven pendulum with an explicit time-dependent torque
F(t) = A cos(ω_d t). The action depends explicitly on t, so the
time-translation symmetry is broken and Noether predicts E is no
longer conserved. Here a symplectic integrator no longer buys you
anything because the system is non-autonomous; I used RK4.

Fourth, a particle in an asymmetric 1D potential
V(x) = 0.5 x² + 0.1 x³ + 0.05 x⁴. The cubic term breaks the
left-right (translation) symmetry of the well. Noether says linear
momentum should not be conserved. The quartic is just there to keep
the particle from running off to infinity so the curves are
plottable.

I wrote everything in one ~200-line Python file. No PDE solver, no
clever integrator library. The whole point was to keep the moving
parts visible.

## What happened

The Kepler orbit is the clean case. After 20 full periods, the
relative energy drift is 7.4 × 10⁻⁶, and that drift is *bounded*: the
trajectory of E vs t looks like a tiny periodic ripple, not a
runaway. The relative angular-momentum drift is 5.2 × 10⁻¹⁴. That is
not the integrator being clever about L. It is the fact that the
central-force update conserves r × v identically to machine
precision, and the ripple I am seeing is double-precision round-off
adding up over about 1.3 × 10⁵ steps.

The undriven pendulum behaves similarly. Relative E drift is
2.3 × 10⁻⁷ over t = 200 (roughly 32 periods at this amplitude).
Bounded ripple again. The pendulum is anharmonic at this amplitude,
so its period is not the small-angle 2π / sqrt(g/l); the integrator
nevertheless tracks the right invariant orbit.

The driven pendulum is the interesting one. With A = 0.5 and
ω_d = 0.8 (chosen to sit roughly on resonance, not exactly), the
energy walks around its initial value and the *range* of E over the
run is roughly 193 times |E_0|. That is the driver doing work on the
system and then taking it back, but not by equal amounts. The action
S = ∫ L dt has explicit t-dependence through cos(ω_d t), so by
Noether there is no time-invariance and no conserved Hamiltonian.
The integrator does not "lose" energy; it correctly reports that
energy is not a constant of motion in this system.

Here is the heart of the symplectic step for the Kepler orbit,
stripped of arrays and bookkeeping:

```python
def accel(r):
    rn = np.linalg.norm(r)
    return -GM * r / rn**3

a = accel(r)
for i in range(n_steps):
    v_half = v + 0.5 * dt * a       # half-kick
    r      = r + dt * v_half        # drift
    a      = accel(r)
    v      = v_half + 0.5 * dt * a  # half-kick
```

Three lines of update. The half-kick / drift / half-kick layout is
what makes the scheme symplectic. Switching to plain forward Euler
on the same trajectory produces an energy drift of order 10⁰ over
the same window, so the orbit spirals out. Same physics, same dt,
much worse arithmetic.

For the asymmetric well, momentum p(t) sweeps over a range of 1.25
in natural units while total energy stays put at 2.3 × 10⁻⁷ drift
(symplectic again, because the potential is time-independent). So in
this run E is conserved, p is not. That is Noether read sideways:
remove translation symmetry but keep time symmetry, and you keep
energy and lose momentum.

A compact summary:

| scenario                    | symmetry left   | conserved | drift            |
|-----------------------------|-----------------|-----------|------------------|
| (a) Kepler                  | time, rotation  | E, L      | 7.4e-6, 5.2e-14  |
| (b) Pendulum                | time            | E         | 2.3e-7           |
| (c) Driven pendulum         | none (time off) | none      | ~193 (range/E0)  |
| (d) Asymmetric V(x)         | time            | E only    | p range 1.25     |

![conservation](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/91_conservation.png)

The figure plots all four diagnostics on a single sheet. The top-left
panel is the one I keep coming back to: a perfectly flat angular
momentum line at the floor of double precision, with energy as a
narrow horizontal band an order of magnitude or two above. Both are
artefacts not of physics but of the integrator agreeing with the
theorem.

## What it may mean

The numerical experiments do not "prove" Noether. They confirm the
mechanics in four specific systems and, more usefully, show what
goes wrong when each assumption is dropped. The relevance of this
hands-on view, at least to me, is that the theorem reframes
conservation laws as facts about symmetries of the action rather
than as separate empirical regularities. Energy is not a
fundamental thing you measure; it is the thing that is conserved
*because* the laws do not change with time. If tomorrow the laws
drifted, energy would drift too, and that is exactly what the driven
pendulum makes visible.

The same logic stretches all the way to gauge theories: U(1)
invariance of the QED Lagrangian gives charge conservation,
SU(3) gives the color analogue. Approximate symmetries give
approximately conserved quantities (isospin, hypercharge,
quasi-momentum in a crystal). The driven pendulum is a kindergarten
version of cosmology's puzzle about energy in an expanding
spacetime, where time-translation symmetry is also missing and
"energy conservation" needs careful framing.

Where the simple picture seems to creak, on this kind of toy
problem, is when symmetry is approximate or only present in a
subspace. The asymmetric well still conserves something (energy);
it just isn't momentum any more. Real systems are usually in this
middle ground, and a lot of physics is the work of figuring out
which slice of the symmetry group is intact and which is not.

## Loose ends

The driven pendulum was integrated with RK4, which is not symplectic
and adds its own (small) numerical drift on top of the physical
non-conservation. A cleaner test would split the system into an
autonomous symplectic part plus a forcing term and use an extended-
phase-space symplectic scheme. The asymmetric potential is bounded
by hand; an open V(x) = a x² + b x³ would let me watch p drift
linearly until the particle escapes. With another week, the next
piece would be a gauge analogue: a charged particle in a slowly
varying vector potential, where breaking U(1) gives a calculable
charge drift instead of a momentum one. The code took an evening; the
theorem is from 1918 and still pulls its weight.
