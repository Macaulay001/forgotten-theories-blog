# I counted Dyck paths, parens, binary trees, and polygon triangulations for n up to 10. Catalan's 1844 sequence matched all four.

*Part 77 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1844 Eugene Charles Catalan, a Belgian mathematician then teaching in Paris, published a short note in Liouville's *Journal de Mathematiques Pures et Appliquees* about a sequence

    C_n = (1 / (n+1)) * C(2n, n)

= 1, 1, 2, 5, 14, 42, 132, 429, 1430, 4862, 16796, ...

He was not the first to write these numbers down. Euler and Segner had worked with them in the 1750s while counting how many ways you can chop a convex polygon into triangles using non-crossing diagonals. Catalan's contribution was to study the recurrence and to connect the sequence to a different problem (well-formed bracket strings). The name "Catalan number" got attached later. What stayed surprising for a long time, and what still feels a little improbable when you write the enumeration out by hand, is how often this same integer sequence shows up in combinatorics problems that do not obviously have anything to do with each other.

I had read the list. I had never enumerated all four families on a laptop and watched the counts collide.

## What I tried

The plan was to pick four classical Catalan-counted families, write a separate brute-force enumerator for each, and then compare the counts against the closed form C_n = binomial(2n, n) / (n+1) for n = 0 through 10. Four families, four pieces of code, none of which knows about C_n internally.

The four families I picked:

1. Dyck paths of length 2n. Lattice paths that start at (0,0), end at (2n,0), step by either (+1,+1) (an up-step) or (+1,-1) (a down-step), and never dip below the x-axis. I enumerated by iterating over all 2^(2n) sign sequences of length 2n and filtering for the prefix-sum constraint.
2. Balanced parenthesis strings of length 2n. Strings of "(" and ")" where every prefix has at least as many "(" as ")", and the whole string is balanced. Same brute force: iterate over 2^(2n) candidate strings, filter.
3. Plane rooted binary trees with n internal nodes. Each internal node has a left and a right subtree. Build them recursively: for n internal nodes, pick k in {0,...,n-1} to put in the left subtree and (n-1-k) on the right, recurse, take all combinations. This recursion does not use the Catalan recurrence numerically; it just splits structurally.
4. Triangulations of a convex (n+2)-gon by non-crossing diagonals. Recurse on the base edge (0, n+1): every triangulation contains exactly one triangle that uses this edge, so pick its apex vertex k in {1, ..., n}, recurse on the two subpolygons, glue the triangle sets together. Deduplicate at the end by storing each triangulation as a frozenset of triangle triples.

The first two get expensive fast (2^20 = 1.05e6 candidates at n = 10, 2^22 at n = 11), but n = 10 fits in a few seconds. The recursive enumerators for trees and triangulations stay much smaller because they only generate valid objects.

The check is just whether the four counts agree with C_n at every n. They either do, exactly, or they don't.

## What happened

They did. Every single row.

```python
# verify all four enumerators against the closed form
for n in range(11):
    cn = math.comb(2*n, n) // (n + 1)
    d = len(enum_dyck(n))
    p = len(enum_paren(n))
    b = len(enum_btrees(n))
    t = len(enum_triangulations(tuple(range(n + 2))))
    assert d == cn == p == b == t, (n, cn, d, p, b, t)
    print(n, cn, d, p, b, t)
```

Output for n = 0 through 10:

| n | C_n | Dyck | parens | trees | triangulations |
|---|------|------|--------|-------|----------------|
| 0 | 1 | 1 | 1 | 1 | 1 |
| 3 | 5 | 5 | 5 | 5 | 5 |
| 4 | 14 | 14 | 14 | 14 | 14 |
| 6 | 132 | 132 | 132 | 132 | 132 |
| 8 | 1430 | 1430 | 1430 | 1430 | 1430 |
| 10 | 16796 | 16796 | 16796 | 16796 | 16796 |

(Full table in the figure.) Four independent counters, four columns of identical integers. The Dyck enumerator filters 1,048,576 candidate sign strings at n = 10 and keeps exactly 16,796 of them. The parenthesis enumerator filters 1,048,576 candidate strings at n = 10 and keeps exactly 16,796 of them. The recursive binary-tree builder produces 16,796 plane trees. The triangulation enumerator returns 16,796 distinct triangle sets for the 12-gon.

The second check is the asymptotic. There is a classical result that

    C_n ~ 4^n / (n^(3/2) * sqrt(pi))

as n grows. This falls out of Stirling's approximation applied to the binomial. Using the closed-form C_n (the enumeration becomes infeasible past n ~ 12, but the formula is cheap):

| n | C_n / (4^n / (n^(3/2) sqrt(pi))) |
|---|----------------------------------|
| 5 | 0.81 |
| 10 | 0.90 |
| 15 | 0.93 |
| 20 | 0.95 |
| 25 | 0.96 |
| 30 | 0.96 |

The ratio creeps up toward 1 from below. It does not approach particularly fast (the next correction term is order 1/n), but the leading 4^n / (n^(3/2) sqrt(pi)) does set the right scale, and on this sample the formula recovers C_30 = 3,814,986,502,092,304 to within about 4 percent.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/77_catalan_summary.png)

The figure shows one Dyck path of length 8, all five balanced parenthesis strings at n = 3, one binary tree on three internal nodes, one of the fourteen triangulations of a hexagon, the full count table, and the asymptotic ratio plot.

## What it may mean

What I find quietly striking about the experiment is not that the counts agree (the bijections are known and the proofs are textbook), but how cheaply the agreement is visible. The Dyck path and the parenthesis string are obviously the same object under the substitution "(" -> +1, ")" -> -1. The bijection between Dyck paths and binary trees is a small recursion: read the path until the first time it returns to zero, recurse on the bit before the return as the left subtree, recurse on the bit after as the right subtree. The bijection between binary trees and triangulations is older and slightly more involved: trace the polygon, label internal triangles by the order they get cut off.

None of these bijections requires that you compute C_n at all. They show that the four families have the same size by exhibiting structural maps between them. The closed form C_n = binomial(2n, n) / (n+1) is then almost a separate result: it tells you what that common size happens to equal.

The pattern of one integer sequence counting many things keeps recurring in mathematics. Pascal's triangle does it. Bell numbers, Motzkin numbers, Schroder numbers all have lists of "what they count" that run into the dozens. Richard Stanley's *Enumerative Combinatorics* dedicates an exercise to listing 66 different Catalan-counted families. The reason this happens, I think, is that many natural recursions have the same shape (split into two pieces, take all combinations, sum), and that shape forces the same generating function, and the same generating function forces the same coefficients.

## Loose ends

A few honest caveats. The brute-force Dyck and parenthesis enumerators are exponential in n; n = 10 is already a million candidates and n = 14 would be a quarter-billion. The recursive enumerators for trees and triangulations are roughly linear in the output size, so they would scale further, but I capped at n = 10 to keep all four columns directly comparable. The triangulation enumerator deduplicates by storing each triangulation as a set of triangles; I trust this only because the answers come out correct, not because I proved uniqueness independently. With another afternoon I would implement the rotation-distance metric on triangulations (the operation that turns one triangulation into another by flipping a diagonal) and see how the resulting graph on the 14 hexagon triangulations compares to the associahedron, which is the canonical polytope whose vertices are the triangulations of an (n+2)-gon.
