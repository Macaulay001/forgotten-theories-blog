# I located the first 30 zeros of Riemann's zeta on the critical line. They all sat exactly where 1859 said they would.

*Part 95 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In November 1859, Bernhard Riemann submitted an eight-page memoir to the Berlin Academy. The paper was about prime numbers. Buried in the middle, almost as an aside, was a sentence that has shaped analytic number theory ever since: it is very likely that all the non-trivial zeros of the function ζ(s) lie on the line where the real part of s equals 1/2. Riemann said he had checked a few and moved on. He never published a proof. The conjecture has resisted everyone since, including Hardy, Hilbert, Selberg, and a long parade of computational projects that have ground out the first ten trillion zeros without finding a single one off the line. It is now one of the seven Clay Millennium Problems with a million-dollar prize attached.

I wanted to see the thing with my own arithmetic. Not to settle anything (no laptop will), but to actually compute Riemann's first claim and watch zeta vanish at the right altitudes.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/95_Z_function.png)

## What I tried

The zeta function is awkward on the critical line. The defining Dirichlet series 1 + 2⁻ˢ + 3⁻ˢ + ... only converges for Re(s) > 1, and the analytic continuation needed to evaluate ζ at s = 1/2 + it involves either the functional equation or Riemann's own asymptotic expansion (the Riemann-Siegel formula, rediscovered by Siegel in Riemann's notebooks in 1932). Worse, |ζ(1/2 + it)| oscillates and never quite touches zero in a numerically clean way unless you have enough digits of precision.

The standard trick is to use Hardy's Z-function. It is a real-valued function of a real variable t, defined so that |Z(t)| = |ζ(1/2 + it)| and Z(t) changes sign exactly where ζ has a zero on the critical line. So finding zeros of zeta on the line reduces to finding sign changes of an ordinary real function. That part is high school.

I used `mpmath` at 30 decimal digits of precision. The `mpmath.siegelz` call evaluates Z(t) via the Riemann-Siegel expansion. I scanned t from 0 to about 105 in steps of 0.5, watched for sign changes, then refined each crossing with `mpmath.findroot` using the Anderson solver, falling back to bisection. After locating each ordinate I evaluated `mpmath.zeta(0.5 + 1j t)` as an independent sanity check: if the located t is a real zero of zeta on the line, |ζ| there should be tiny.

The reference values are well-tabulated. The first few are 14.1347, 21.0220, 25.0109, 30.4249, 32.9351, going up to 101.3179 by the thirtieth. Odlyzko's tables list these to fifty-plus digits. So this is a comparison, not a fishing expedition.

## What happened

All thirty located ordinates matched the published values. The largest discrepancy across the first 30 zeros was about 4.8 × 10⁻¹³, which is at the floor of double-precision rounding when reading the published ordinates as floats. The largest |ζ(1/2 + it)| at any located ordinate was 2.2 × 10⁻¹⁴. In other words, my located ordinates are zeros of ζ on the critical line to within numerical noise, and they sit at the heights Riemann's successors said they would.

```python
import mpmath as mp
mp.mp.dps = 30

zeros = []
ts = [0.5 * i for i in range(0, 211)]
for a, b in zip(ts[:-1], ts[1:]):
    if mp.siegelz(a) * mp.siegelz(b) < 0:
        root = mp.findroot(mp.siegelz, (a, b), solver="anderson")
        zeros.append(float(root))
        print(root, abs(mp.zeta(mp.mpc(0.5, root))))
```

That fits in a notebook cell. It produces the first ~30 zeros in under a minute on one core.

A small slice of the comparison table:

| n | t (computed) | t (published) | |Δ| |
|---|---|---|---|
| 1 | 14.1347251417 | 14.1347251417 | 1.8e-15 |
| 5 | 32.9350615877 | 32.9350615877 | 0 |
| 11 | 52.9703214777 | 52.9703214777 | 7.1e-15 |
| 26 | 92.4918992706 | 92.4918992706 | 1.4e-14 |
| 29 | 98.8311942182 | 98.8311942182 | 4.8e-13 |

The last column drifts toward 10⁻¹³ at the higher ordinates, which is mostly because the published values I compared against were truncated to 15 digits in my table.

I also rendered log10 |ζ(s)| over the slice σ in [0, 1], t in [0.5, 50]. The image shows dark vertical wells aligned with σ = 0.5. Outside the critical line there is no comparable depth in this window. The picture is not a proof of anything (there could be off-line zeros elsewhere), but it is a striking visualization of why people believe the conjecture: at the resolution my screen can show, the zeros of zeta in this strip really do hug a single vertical line.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/95_zeta_heatmap.png)

The Z-function plot is the cleanest part of the exercise. Z(t) wiggles, crosses zero, wiggles again, and never misses a crossing in [0, 105]. Each crossing maps to a known zero. Each one falls on the critical line because Hardy's Z is, by construction, a real-line story about zeros on Re(s) = 1/2. (If a zero existed off the line, Z would not see it; you would have to use a different machinery to detect that.)

## What it may mean

This is consistent with what Hardy proved in 1914 (infinitely many zeros lie on the critical line) and with what large-scale computation has been telling us for decades. Van de Lune, te Riele and Winter (1986) verified the first ~1.5 × 10⁹ non-trivial zeros. Gourdon (2004) pushed the count past 10¹³. Platt and Trudgian (2021) gave a rigorous bound on RH's truth up to height 3 × 10¹². My contribution here is, frankly, a postcard: thirty zeros, in a couple of minutes, on a laptop, matching values that have been known since at least the 1980s. The point is that you, the reader, can replicate this in an hour.

What may matter more than the numbers is what the picture suggests. The zeros are not randomly arranged. Their spacing statistics, when you compute many of them, match the gap distribution of eigenvalues of large random Hermitian matrices (the GUE statistics noticed by Montgomery and Dyson in 1973). That coincidence may turn out to be the real opening in the wall. If zeta's zeros are eigenvalues of some self-adjoint operator we have not yet found, the Hilbert-Polya philosophy would force them all onto the line. We do not have that operator. We have, however, a lot of suggestive numerics.

I should hedge plainly: nothing here proves the hypothesis. The first 10¹³ zeros being on the line is not a guarantee that the 10¹³ + 1st is. Odlyzko has noted that some pathological behavior of zeta only kicks in near heights like 10³⁰, far beyond any computation. Still, the consistency of the numerics is the kind of fact that keeps the conjecture believed.

## Loose ends

A few honest caveats. My check uses `mpmath.siegelz` as a black box; I did not re-derive the Riemann-Siegel formula or audit its truncation error at high t. The comparison to "known" values is limited by the precision of the table I pasted in. And I only looked at the first 30 zeros, which is roughly the territory Riemann himself had access to. With another week I would push to the first 100,000 zeros, check pair correlations against the Montgomery-Dyson GUE prediction, and look for any sign that the gap statistics drift as t climbs. The code to do this is short; the patience required is what I am missing tonight.

You can rerun the whole thing with `pip install mpmath matplotlib numpy` and the script in the repository. It takes about two minutes.
