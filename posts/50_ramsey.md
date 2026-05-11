# I checked all 32,768 two-colorings of K_6. Every single one contains a monochromatic triangle.

*Part 50 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1930 Frank Ramsey, then 26 and a year from death, wrote a paper called "On a problem of formal logic". Buried inside it was a combinatorial lemma that he treated as a side technicality. The lemma said that for any two positive integers r and s, there is a smallest number R(r, s) such that any party of R(r, s) people will contain either r mutual friends or s mutual strangers. The smallest interesting case is R(3, 3) = 6. In a group of six people, you are guaranteed three who all know each other or three who are pairwise strangers. In a group of five, you can arrange things so neither holds.

The graph-theoretic version is cleaner. Take the complete graph on n labeled vertices, color each edge red or blue however you like, and ask whether some triangle has all three edges the same color. Ramsey's claim, on this small case, is that at n = 6 you cannot avoid it, and at n = 5 you can. The proof in any combinatorics textbook is two paragraphs of pigeonhole. I had taught it. I had never actually checked it by brute force.

## What I tried

The plan was the most literal possible reading of the statement. K_5 has C(5, 2) = 10 edges, so there are 2^10 = 1,024 distinct red-blue colorings. K_6 has C(6, 2) = 15 edges, so 2^15 = 32,768 colorings. For each one I would loop through all C(n, 3) triangles, check whether all three of their edges share a color, and record whether the coloring was triangle-free in both colors.

If Ramsey's count is right, the K_5 enumeration should produce a nonzero count of triangle-free colorings, including the famous pentagon-pentagram pattern (red edges forming the convex pentagon 0-1-2-3-4-0, blue edges forming the inscribed five-pointed star 0-2-4-1-3-0). The K_6 enumeration should produce zero. That is the entire test.

Both enumerations fit in a couple of seconds on one laptop core. There is no statistics here, no Monte Carlo, no sampling noise. Exhaustion either confirms the claim across every configuration or finds a counterexample, in which case Ramsey was wrong and a great deal of mid-century combinatorics would need rewriting.

The pentagon-pentagram coloring deserves a moment of its own. The pentagon and pentagram share no edge, so coloring the pentagon red and the pentagram blue partitions all 10 edges of K_5. Any triangle on 5 vertices uses three of those edges, and a quick check shows it always uses at least one pentagon edge and at least one pentagram edge. The triangle therefore cannot be monochromatic. This single explicit construction is what keeps R(3, 3) strictly greater than 5.

```python
# brute-force check of every 2-coloring on K_n
for bits in range(1 << len(edges)):
    color = [(bits >> i) & 1 for i in range(len(edges))]
    mono = False
    for a, b, c in triangles:
        if color[idx[a,b]] == color[idx[b,c]] == color[idx[a,c]]:
            mono = True
            break
    if not mono:
        triangle_free.append(bits)
```

That, plus an edge-index map, is the whole simulator.

## What happened

The K_5 enumeration produced 12 triangle-free colorings out of 1,024. The pentagon-pentagram pattern (bit-pattern 665 in the lexicographic edge order) was in the list. The other 11 are its symmetry copies: rotations of the pentagon by 1, 2, 3, and 4 positions, plus reflections, plus the dual where you swap the role of red and blue. There is one unlabeled triangle-free coloring of K_5, and the labeled enumeration sees it 12 times.

The K_6 enumeration produced exactly zero triangle-free colorings out of 32,768. Every single arrangement of 15 red-or-blue edges on six vertices forces some triangle to be monochromatic. Ramsey's lemma holds across the entire exhaustive table.

| graph | edges | total colorings | triangle-free |
|-------|------:|----------------:|--------------:|
| K_5   | 10    | 1,024           | 12            |
| K_6   | 15    | 32,768          | 0             |

![Pentagon-pentagram coloring of K_5](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/50_k5_pentagram.png)

The K_5 figure above shows one of the 12. Solid red edges form the outer pentagon, dashed blue edges form the inscribed star. You can pick any three vertices and trace the triangle they form: it will always include at least one of each color.

The K_6 case rewards a closer look. I generated one random coloring of K_6 and asked the program where the forced monochromatic triangle lay. The answer, on seed 7, was a blue triangle on vertices (0, 3, 4). The textbook proof tells you why this has to happen for any seed. Pick any vertex v. It has 5 incident edges. By pigeonhole, at least 3 of them share a color, say red, going to neighbors a, b, c. If any of the edges (a, b), (a, c), or (b, c) is also red, that red edge plus v gives a red triangle. If all three of (a, b), (a, c), (b, c) are blue, then a, b, c form a blue triangle. Either way, monochromatic triangle. The exhaustion just confirms the structural argument across every seed at once.

![Random K_6 coloring with its forced monochromatic triangle](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/50_k6_forced_triangle.png)

A few things in this exercise stood out. One was how cheap the verification was. A 1930 result whose proof requires the pigeonhole step plus a case analysis falls out of two nested loops in under a second on hardware that did not exist when Ramsey was alive. The proof did real intellectual work in 1930; the verification is now a warm-up exercise.

The second was how thin the margin is. K_5 admits exactly one triangle-free pattern (up to relabeling). Add a single vertex and that pattern cannot be extended. The pentagon-pentagram coloring of K_5 sits on a knife's edge of combinatorial impossibility. Larger Ramsey numbers behave the same way. R(4, 4) = 18 has exactly one critical Ramsey graph on 17 vertices (the Paley graph of order 17). R(5, 5) sits somewhere in [43, 46] and the critical configurations, if they exist, are still unknown.

## What it may mean

Ramsey theory is the formal study of order forced out of size. Once a structure is large enough, certain substructures appear regardless of how you color, label, or arrange the edges. The principle generalizes far beyond graphs. Van der Waerden's theorem says that any 2-coloring of the integers up to V(k) contains a monochromatic arithmetic progression of length k. Schur's theorem says any coloring of the integers contains a monochromatic solution to x + y = z. Hindman's theorem says the same for finite sums of any subset. All of these are descendants of Ramsey's lemma, and all share the same flavor: structure is unavoidable once the size threshold is crossed.

The exact thresholds remain stubbornly hard. R(3, 3) = 6 is the only Ramsey number where exhaustion is easy. R(4, 4) = 18 took a long search. R(5, 5) is famously the smallest open case, with bounds 43 and 46 that have barely moved since the 1990s. Paul Erdős liked to say that if aliens demanded the value of R(5, 5) under threat of destroying Earth, humanity should pool all its computers and mathematicians. If they demanded R(6, 6), humanity should try to destroy the aliens first.

The brute-force approach used here scales as 2^(n(n-1)/2). At n = 6 that is 32,768 and instant. At n = 17 it is 2^136, well beyond any conceivable computation. R(5, 5) cannot be resolved by enumeration. It will need either a clever construction that pushes the lower bound past 45, or a structural proof that rules out the existence of certain critical graphs above some size.

## Loose ends

With another week I would generate the full list of unlabeled triangle-free K_5 colorings (there is only one) and write a small isomorph-rejection routine to confirm that the 12 labeled instances really do collapse to a single orbit under the action of S_5 cross color-swap. I would also push the enumeration to K_8 with target clique size 4, which is 2^28 colorings and around 270 million configurations, looking for the Paley(17) analog and confirming R(4, 4) = 18 along the way. The pipeline replicates in under a minute on any laptop, and the conclusion of the small case has been stable for 96 years.
