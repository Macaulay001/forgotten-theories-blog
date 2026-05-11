# Kirchhoff's 1845 student paper reduces any resistor maze to ten lines of Python

*Part 93 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Gustav Kirchhoff was 21 and still a student in Konigsberg when he
wrote down the two rules that bear his name. At every node, the
algebraic sum of currents is zero. Around every closed loop, the
algebraic sum of voltage drops is zero. Combined with Ohm's law,
those two statements pin down a unique solution for any network of
linear resistors and sources, no matter how tangled the topology.
The paper is short and the argument is almost embarrassing in its
simplicity. Conservation of charge. Conservation of energy.
Linearity. Done.

What makes the laws interesting in 2026 is not whether they hold,
they have been audited for 180 years, but what they reduce to once
you write them in matrix form. The whole edifice collapses to
Y v = i, where Y is a sparse n×n admittance matrix you can build
from an edge list in a single loop. That is the form every SPICE
simulator on Earth solves under the hood. I wanted to feel how
small the working code really is.

## What I tried

The plan was three test cases, each with a known answer.

The Wheatstone bridge, first. Four resistors in a diamond, a
galvanometer across the middle, a battery at the corners. The
balance condition R1/R2 = R3/R4 is supposed to drive the
galvanometer current to exactly zero, regardless of the
galvanometer's own resistance. This is the trick that let Charles
Wheatstone (and before him Samuel Hunter Christie) measure unknown
resistances to four or five decimals in the 1830s, using an eye
watching a needle.

The symmetric infinite ladder, second. The setup is a chain of
identical "rungs", each one a 1 Ω series resistor followed by a
1 Ω shunt to ground. What is the input resistance as the
chain gets long? There is a classic recursion: R_{N+1} = 1 +
1·R_N/(1+R_N). The fixed point is the positive root of
x² − x − 1 = 0, which is the golden ratio φ = 1.6180339887... R.
Some puzzle collections quote (1+√3)/2 R ≈ 1.366 R for "the
infinite ladder"; that value comes from a slightly different
topology (shunt-first, or unequal element values). For the
canonical 1-1 ladder, φ is the right answer, and I wanted to watch
my linear solve walk up to it.

The N×N square grid, third. Unit resistors on every edge. Two
classical limits exist for the infinite 2D lattice: the effective
resistance between adjacent nodes is exactly 1/2 R (a one-line
superposition argument), and between diagonal nodes across one
square is 2/π R ≈ 0.6366 R, a Green's-function result. Both should
emerge as I increase N and look near the center, away from the
boundary.

The whole pipeline is one admittance assembler shared across all
three problems. For each edge (i, j, R), you add 1/R to Y[i,i]
and Y[j,j], and subtract 1/R from Y[i,j] and Y[j,i]. Then pick a
reference node, delete its row and column, and solve. To get an
effective resistance between two nodes, inject +1 A at one of
them and ground the other.

## What happened

Kirchhoff held up at floating-point precision in every case.

The Wheatstone solver returns I_galv = 0.0 A exactly at the
balance point R4 = R2·R3/R1 = 100 Ω, and a straight line through
zero as R4 is swept by ±20 Ω with a 5 V supply. The slope is
about 4.7 µA per ohm of imbalance for this geometry. The current
through the galvanometer comes from a 1 kΩ branch in parallel with
the two divider arms; the divider potential difference at
imbalance shows up undiminished in the matrix solve. Nothing
exotic, but it is satisfying that the laws give 0.000e+00 A and
not 1e-17 A.

The ladder convergence is sharper. The Y-solve and the analytical
recursion agree to ten decimals at every N, which is a useful
self-consistency check (different code paths, same answer). The
gap to φ shrinks geometrically:

| N | R_in | |R_in − φ| |
|---|---|---|
| 3 | 1.625000 | 6.9 × 10⁻³ |
| 5 | 1.618182 | 1.5 × 10⁻⁴ |
| 10 | 1.6180339985 | 9.7 × 10⁻⁹ |
| 20 | 1.6180339887 | < 10⁻¹⁰ |
| 200 | 1.6180339887 | < 10⁻¹⁰ |

By N = 20 the answer is φ to all the digits double precision can
carry. The rate works out to about a factor of φ⁻² ≈ 0.382 per
added rung, the classical contraction ratio of the recursion.

Here is the core, with no boilerplate:

```python
def build_admittance(edges, n_nodes):
    Y = np.zeros((n_nodes, n_nodes))
    for i, j, R in edges:
        g = 1.0 / R
        Y[i, i] += g; Y[j, j] += g
        Y[i, j] -= g; Y[j, i] -= g
    return Y

def R_effective(edges, n_nodes, a, b):
    Y = build_admittance(edges, n_nodes)
    keep = [k for k in range(n_nodes) if k != b]
    i_vec = np.zeros(len(keep)); i_vec[keep.index(a)] = 1.0
    v = np.linalg.solve(Y[np.ix_(keep, keep)], i_vec)
    return v[keep.index(a)]
```

That is the whole circuit simulator for resistors. Ten lines.

The grid is where it gets a little more textured. With N up to
17 (so 289 nodes, dense solve, well under a second on a laptop),
the adjacent-node resistance near the center is 0.5019 Ω,
shrinking toward 1/2 from above. The diagonal-across-one-square
value is 0.6405 Ω at N = 17, closing on 2/π = 0.6366 from above.
The boundary contaminates both estimates: a node near the edge of
the lattice "sees" fewer return paths than a node deep in the
bulk, so finite-N values overshoot the infinite-lattice limit.

A short Aitken Δ² acceleration on the diagonal sequence brings
the last three N values to about 0.6419 Ω, which is closer to
2/π but not quite there. The convergence is only 1/N, not
geometric, which is why even N = 17 is still in the third
significant figure. Corner-to-corner resistance, by contrast,
keeps growing with N (≈ 3.5 Ω at N = 15) and has no finite limit:
the 2D lattice Green's function diverges logarithmically.

![grid](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/93_kirchhoff_summary.png)

Three panels: the Wheatstone null, the ladder marching to φ on
a log axis, and the grid's two well-known limits. Nothing
surprising, and that is the point.

## What it may mean

Two small things worth noticing, neither of them new.

The first is how much of "circuit theory" really lives inside a
single sparse linear solve. Once you have an admittance assembler
and a reference-node convention, the rest is bookkeeping. The
admittance matrix is symmetric (passive networks), positive
definite after grounding (no zero modes), and sparse for any
physical layout. Modern simulators add complex Y for AC analysis,
nonlinear elements via Newton iteration, and time-stepping for
transients, but the core data structure is the one Kirchhoff
implicitly described in 1845.

The second is the way irrational numbers fall out of resistor
networks. The golden ratio appears in the symmetric ladder. The
quantity 2/π appears in the 2D grid diagonal. The corner-to-corner
grid resistance grows logarithmically and is related to digamma
values. These are not coincidences; they are the lattice Green's
functions of the discrete Laplace operator, which is what Y
becomes on a uniform grid. The same Green's functions appear in
random walks and in percolation theory. The link between resistor
networks and random walks (Doyle and Snell, 1984) is one of the
prettier corners of probability.

A historical note. Kirchhoff's 1845 paper does not contain the
matrix formulation. That came much later, after the language of
linear algebra had matured. Reading the original is interesting
mostly for what it leaves implicit. He talks about loops and
nodes as if anyone would obviously count them; modern textbooks
fuss about how to pick an independent loop basis. The 21-year-old
just knew.

## Loose ends

The grid extrapolation could be pushed further with a sparse
solver. With csr_matrix and scipy.sparse.linalg.spsolve, N = 50
to 100 is easy, and the 2/π limit would resolve to four digits.
A fitted form of the boundary correction (~ a + b/log(N) for
adjacent, ~ a + b/N for diagonal) would let me extrapolate
cleanly. Nonlinear elements and AC analysis are the obvious next
extensions; both fit inside the same Y framework with a single
function call per Newton step. I did not run them. The full
pipeline replicates in under a minute on a laptop and the NPZ is
in the repo.
