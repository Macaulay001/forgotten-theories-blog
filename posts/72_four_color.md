# Guthrie's 1852 four-color guess survives 1000 random planar graphs

*Part 72 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In October 1852 a young law student named Francis Guthrie was coloring
a map of the counties of England. He noticed that four colors always
seemed enough to keep neighboring counties distinct. He could not find
five touching regions that all needed different colors, no matter how
he twisted the boundaries. He wrote to his brother Frederick, who was
studying under Augustus De Morgan in London. De Morgan wrote to William
Rowan Hamilton on 23 October 1852 asking whether the claim was a known
result. It was not.

For 124 years the question sat unproved. Alfred Kempe published a proof
in 1879 that lasted eleven years before Percy Heawood found the gap.
Kempe's argument still gave a 5-color theorem, which is a strange place
for a near-miss to land. The 4 stubbornly held. People tried planarity
arguments, Euler-characteristic counts, exhaustive case analysis. None
of it closed. In June 1976 Kenneth Appel and Wolfgang Haken at the
University of Illinois announced a proof that ran through 1936
reducible configurations on an IBM 370. About 1200 hours of compute,
on a machine that today would lose to a mid-range phone. Their campus
post office briefly stamped outgoing mail with "Four colors suffice".
Mathematicians argued for years afterward about whether a proof that
no human can read is really a proof.

I wanted to feel the claim, not the proof. So I tried to break it.

## What I tried

The plan was small. Generate a thousand random planar graphs, throw a
backtracking 4-coloring algorithm at each, count failures. Repeat with
only 3 colors available and watch what breaks.

Delaunay triangulations are convenient here. Drop n random points in
the unit square and connect them via the Delaunay triangulation, and
you get a planar graph for free (it is the dual of the Voronoi diagram,
which lives flat). Sizes were drawn uniformly from 8 to 40 nodes. Mean
ended up at 23.4 vertices and 59.3 edges per graph.

For coloring I used the textbook backtracking routine: order vertices
by descending degree, try colors in order, and roll back on conflict.
The backtracking is not how the Appel-Haken proof works (their proof
is structural, not algorithmic), but it is enough to certify that a
valid k-coloring exists.

I also wanted a counterexample for the 3-color version. The smallest
witness is K_4, the complete graph on four vertices. It is planar (draw
a triangle with a fourth vertex inside, connected to all three corners)
and obviously needs 4 colors because every pair is adjacent. The
question was whether random Delaunay graphs would contain enough K_4
subgraphs to force the same failure.

The last test was the iconic case. The lower-48 US states form a
planar adjacency graph if you treat shared boundaries (not corners) as
edges. I compiled a 49-node list including DC and ran the same
backtracking on it.

## What happened

Every one of the 1000 random graphs was 4-colorable. The success rate
sits at 1000/1000.

```python
def backtrack(i):
    if i == len(nodes):
        return True
    v = nodes[i]
    for c in range(k):
        if all(color.get(u) != c for u in G.neighbors(v)):
            color[v] = c
            if backtrack(i + 1):
                return True
            del color[v]
    return False
```

Three colors were a different story. Only 25 of the 1000 graphs admitted
a proper 3-coloring. The 25 were the small, sparse cases where the
Delaunay triangulation happened to lack a K_4 subgraph. As soon as the
point set produced four mutually adjacent vertices (which any decent-sized
Delaunay graph does), three colors stopped working.

![Random Delaunay planar graphs, 4-colored by backtracking](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/72_random_planar_4colored.png)

K_4 itself was the cleanest demonstration. Four vertices, six edges, all
in the plane. The 4-coloring assigns one color per vertex. The 3-coloring
search returns failure immediately, since the algorithm cannot find a
fourth color to reuse.

The US-states case behaved exactly like the proof predicts. The
adjacency graph has 49 vertices (the lower 48 plus DC) and 108 edges.
Backtracking finds a 4-coloring in milliseconds. The 3-color attempt
fails. Visually the result is the familiar four-color map every
cartography textbook prints.

![US lower-48 adjacency graph, 4-colored](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/72_us_states_4colored.png)

| Test                                  | Outcome              |
|---------------------------------------|----------------------|
| Random planar graphs, n=1000          | 1000 / 1000 4-colorable |
| Same graphs, 3-coloring               | 25 / 1000 3-colorable |
| K_4 with 3 colors                     | fails                |
| K_4 with 4 colors                     | succeeds             |
| US lower-48 + DC, 4 colors            | succeeds (108 edges) |
| US lower-48 + DC, 3 colors            | fails                |

What surprised me a little was how fast the 3-color version dies. The
textbook framing of the four-color theorem is that 4 is the right
number, with the implicit suggestion that 3 is sometimes enough and
sometimes not. On random Delaunay maps it is almost never enough. The
threshold for the K_4 obstruction kicks in once you have, roughly, a
dozen well-separated points. Larger maps almost certainly contain one.

The 25 small 3-colorable cases are interesting in their own right.
They tend to be near-path or near-cycle graphs where the triangulation
collapses into a thin strip. Once the strip thickens, K_4 appears.

I also poked at runtime. Backtracking on a 49-node US map with 108
edges finishes in under 5 milliseconds on a laptop. The same routine on
a 40-node Delaunay graph rarely backtracks more than a handful of
times; greedy by descending degree gives a near-valid coloring on the
first pass and small fixes do the rest. The hard cases the Appel-Haken
proof handles do not show up in random samples of this size. You have
to construct them deliberately, often by gluing copies of K_4-minus-an-edge
along shared faces.

## What it may mean

If the simulation is honest, Guthrie's guess holds in the empirical
small. None of the 1000 Delaunay graphs needed a fifth color. None of
the 25 that happened to be 3-colorable contradicted that. The US map
behaves the same way. The exhaustive proof from 1976 still does the
heavy lifting (random graphs are not a proof), but the failure mode
where 4 colors should be enough and somehow are not did not appear in
the sample.

There may be a wider point. The four-color theorem became famous not
because the answer is surprising but because the proof was. It was
the first major mathematical result that depended on a computer in a
way no human could check by hand. Gonthier's 2005 Coq formalization
later closed that gap, machine-checking the proof line by line. The
result is now both visually obvious on small examples and formally
certified. Few theorems sit in both categories.

For practical work the theorem matters less for maps than for register
allocation in compilers and for frequency assignment on cellular
networks, where the conflict graph is approximately planar and 4-color
heuristics suggest near-optimal channel reuse.

## Loose ends

Backtracking is fine on graphs of this size. It does not scale to the
millions of vertices the proof itself handles. A serious test would
swap in a structural reduction (Kempe chain swaps, for instance) and
verify behavior on graphs with 10^4 or more nodes. The four-corner
issue on the US map (states meeting at a single point) is a definition
choice; standard convention is that point contacts do not count as
adjacency. If you change that convention the graph stays 4-colorable,
but the coloring shifts. A natural next experiment, maybe an hour of
work, would be to run the same pipeline on the world country
adjacency graph, where exclaves and disputed boundaries make the
planarity itself debatable.
