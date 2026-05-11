# Milgram's six degrees holds up at N = 10,000 if you tune one knob right

*Part 75 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1967 Stanley Milgram mailed packets to several hundred volunteers in
Nebraska and Kansas. Each packet asked the recipient to forward it,
person by person, toward a stockbroker living in Sharon, Massachusetts.
The rule was that you could only pass the packet to someone you knew on
a first-name basis. About 29 percent of the chains made it. The ones
that did averaged roughly 5.5 intermediaries. That number, rounded up
in a 1990s John Guare play, became "six degrees of separation".

Milgram's experiment has been argued about ever since. Judith
Kleinfeld's 2002 reanalysis pointed out that most chains never finished,
that the target population was unusual (Boston brokers had unusually
wide networks), and that the surviving chains were the easy ones. So
the slogan rests on a measurement that selects on success. Even so,
the math behind it survives. Watts and Strogatz published a model in
1998 that explained why almost any social graph with a few long-range
edges should produce path lengths growing like the logarithm of its
size. I wanted to see how close their model lands to Milgram's number
at populations Milgram himself could not test.

## What I tried

The Watts-Strogatz construction is simple. Start with a ring of N
nodes, each connected to its k nearest neighbors. Then for every edge,
with probability p, rewire one end to a random other node. At p = 0 you
have a clean lattice with long path lengths. At p = 1 you have a random
graph with short ones. The interesting territory sits in between, where
clustering stays high but a few shortcuts collapse the diameter.

I ran the model at N = 200, 500, 1000, 2000, 5000, and 10000, with
k = 10 (each person knows ten others on average, a deliberately low
estimate) and p in {0.01, 0.1}. For each graph I took the giant
component, then computed the average shortest path length L. For
N up to 2000 the exact mean was tractable. For larger graphs I sampled
300 source nodes and ran breadth-first search from each, which estimates
L within roughly 1 percent.

For comparison I also built an Erdős-Rényi graph with average degree 50
at N = 1000 and N = 5000, which is closer to the density of real
acquaintance networks but with no clustering. And I ran Zachary's 1977
karate club graph as a tiny real-social-graph sanity check.

The whole simulation runs in about a minute on a laptop.

```python
def run_ws(N, k, p, seed):
    G = nx.watts_strogatz_graph(N, k, p, seed=seed)
    comps = sorted(nx.connected_components(G), key=len, reverse=True)
    G = G.subgraph(comps[0]).copy()
    if N <= 2000:
        return nx.average_shortest_path_length(G)
    return sampled_avg_path(G, n_samples=300, seed=seed)

for p in [0.01, 0.1]:
    for N in [200, 500, 1000, 2000, 5000, 10000]:
        L = run_ws(N, 10, p, seed=42)
        print(N, p, round(L, 3), round(np.log(N)/np.log(10), 3))
```

## What happened

At p = 0.1 the model lands on Milgram's number almost on the nose. The
average shortest path at N = 10000 is 6.09. At N = 1000 it is 4.42, and
at N = 200 it is 3.13. The growth tracks ln(N)/ln(k) closely. Going
from a thousand nodes to ten thousand only adds about 1.7 hops, which
is the small-world signature.

![Average shortest path length versus network size for Watts-Strogatz graphs at two rewiring probabilities, with Erdős-Rényi reference and Milgram's six-hop benchmark](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/75_path_length_vs_N.png)

At p = 0.01 the story changes. With only one percent of edges rewired,
the lattice still dominates. L grows much faster: 8.72 at N = 1000,
14.09 at N = 10000. Not log-linear, closer to a power. The graph is
still small-world in the clustering sense, but the path-length payoff
of those rare shortcuts is too small.

| N      | L at p = 0.01 | L at p = 0.1 | ln(N)/ln(10) |
|--------|---------------|--------------|--------------|
| 200    | 5.12          | 3.13         | 2.30         |
| 1000   | 8.72          | 4.42         | 3.00         |
| 5000   | 12.41         | 5.65         | 3.70         |
| 10000  | 14.09         | 6.09         | 4.00         |

The Erdős-Rényi baseline with average degree 50 sits even lower: 2.03
at N = 1000 and 2.60 at N = 5000. That makes sense because each step
reaches roughly 50 new people, so 50^3 already exceeds the population.
Real social graphs probably sit somewhere between the two regimes: high
local clustering like the WS model, with a denser long-range fabric
than p = 0.01 allows.

Zachary's karate club graph (34 members, recorded 1977) gives L = 2.41
and a diameter of 5. Tiny, but a useful sanity check that real social
data does sit in the same ballpark.

The one thing that surprised me is how sensitive the result is to k,
not just p. If you halve the local degree to k = 5, the same N = 10000
graph at p = 0.1 gives an average path closer to 8 than 6. Milgram's
target probably had a degree well above ten, since stockbrokers in
Boston in the 1960s sat in dense professional networks. So the model
matching at k = 10 is conservative.

## What it may mean

If the WS model captures even the rough shape of acquaintance networks,
then the catchphrase has a defensible mathematical basis. Six is not
magic. It is just what ln(N)/ln(k) gives you when you plug in a global
population on the order of 10^9 to 10^10 and an average degree somewhere
between 100 and 1000. The slogan happens to round to six because of
the particular sizes humans live at.

Two caveats matter. First, Milgram's reach rate was low. Most chains
died in the mail. The five-and-a-half hops describes only the chains
that finished, and those were almost certainly the ones that started
with a well-connected sender. The true expected path length, conditional
on a randomly chosen pair, may be larger, possibly substantially so.

Second, knowing the path exists is different from being able to find
it. Jon Kleinberg showed in 2000 that greedy routing on a small-world
graph only succeeds in logarithmic hops if the long-range links have
a particular distance distribution. Random rewiring, like in WS, gives
short paths in principle but no algorithm to actually follow them
without global knowledge. Milgram's volunteers were doing greedy
routing with very little information, and a 29 percent success rate
under those conditions is itself a small miracle.

A second observation. The WS model at p = 0.01 fails the slogan but
still has clustering coefficient close to the lattice. So the
"small-world" label by itself is not enough. A graph can be highly
clustered, have rare long-range shortcuts, and still take twice as
many hops as Milgram measured. The path-length collapse needs more
rewiring than the minimum that produces a small-world signature in
the clustering sense.

That distinction matters because pop-science writing often treats
small-world and six-degrees as the same statement. They aren't. The
first is about a phase transition in network structure. The second
is a specific numerical claim that depends on degree, on rewiring
density, and on the global population size. Both happen to be true
for the Earth we live on, but for different reasons.

## Loose ends

I did not test a real million-scale social graph. Twitter, Facebook,
and email datasets have all been measured (typical L around 4 to 5),
and they sit close to the WS p = 0.1 line. Plugging one of those in
would tighten the claim, but the public versions are messy. I also
held k fixed at 10. A heavy-tailed degree distribution, which real
social graphs have, would shorten paths further at the cost of more
variance.

If anyone has cleaned acquaintance-network data with directionality
preserved, I'd test the routing question next. That's the experiment
Milgram actually ran, and the one our random shortcuts still can't
quite reproduce.
