# I checked Kepler's 1611 claim that F_{n+1}/F_n approaches phi. The match is exact, and the convergence rate hits log(1/phi^2) to six digits.

*Part 78 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In a letter written around 1611, Johannes Kepler pointed out something his contemporaries had mostly overlooked. If you write down the sequence 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144 (each term the sum of the previous two) and form the ratios of consecutive terms (2/1, 3/2, 5/3, 8/5, 13/8, ...), the ratios approach the "divine proportion" that Euclid had described in Book VI of the *Elements*: phi = (1 + sqrt(5)) / 2, about 1.618. Kepler also remarked that the same numbers govern leaf arrangements on plant stems, the so-called phyllotaxis ratios. Leonardo of Pisa (Fibonacci) had introduced the sequence in 1202 as a rabbit-breeding problem. Kepler was the first to connect it to Euclid's golden section.

I have written F_n on chalkboards for years. I had never actually run the convergence test myself and watched the error decay on a log plot. Twenty minutes of code felt overdue.

## What I tried

The plan was three small experiments, each cheap.

First, compute F_1 through F_100 exactly. Python big integers do this without rounding, so the ratios F_{n+1}/F_n are exact rationals up to as far as I want. Float arithmetic saturates at the machine epsilon (~1e-16) by about n=40, which makes the long-tail convergence invisible. To get past that, I held each ratio as a `Fraction` and converted the error |F_{n+1}/F_n - phi| through 80-digit `Decimal` arithmetic. That lets the error decay to 1e-40 cleanly on a log axis.

Second, swap the starting pair. Kepler's claim is really a statement about a recurrence, not about the seeds. So I tried Lucas numbers (2, 1, 3, 4, 7, 11, ...) and five random integer pairs drawn between 1 and 1000. If the math is right, all of them should converge to the same phi, because phi is the dominant eigenvalue of the companion matrix [[1, 1], [1, 0]] regardless of the initial vector (as long as the initial vector has a nonzero component along the dominant eigenvector, which is generic).

Third, perturb the recurrence itself. The tribonacci sequence x_{n+1} = x_n + x_{n-1} + x_{n-2} should converge to a different irrational (the tribonacci constant, around 1.83929). A degenerate recurrence x_{n+1} = 2 x_n - x_{n-1} has a repeated eigenvalue at 1, so ratios should head to 1, not phi. Those are sanity checks that the phi limit is a property of this specific recurrence and not just "any rule of thumb on integer sequences".

The mechanism, written out, is one paragraph. The characteristic polynomial of the Fibonacci recurrence is lambda^2 = lambda + 1, with roots phi ≈ 1.618 and psi = -1/phi ≈ -0.618. The general solution is F_n = A phi^n + B psi^n. The ratio F_{n+1}/F_n is therefore (A phi^{n+1} + B psi^{n+1}) / (A phi^n + B psi^n), which collapses to phi as n grows because |psi/phi| = 1/phi^2 ≈ 0.382 is less than one. The convergence is geometric with rate 1/phi^2 per step. So the error should fall on a straight line with slope log(1/phi^2) ≈ -0.962 on a log scale.

That last number is the part I most wanted to see come out of the simulation.

## What happened

The Fibonacci ratios converge as cleanly as you would hope.

```python
# error in exact 80-digit decimal so we don't saturate at machine eps
from fractions import Fraction
from decimal import Decimal, getcontext
getcontext().prec = 80
PHI = (Decimal(1) + Decimal(5).sqrt()) / 2
fib = [1, 1]
for _ in range(98):
    fib.append(fib[-1] + fib[-2])
err = []
for i in range(1, len(fib) - 1):
    r = Fraction(fib[i + 1], fib[i])
    err.append(abs(Decimal(r.numerator) / Decimal(r.denominator) - PHI))
print(err[-1])  # 9.33e-42
```

At n = 99 the exact error is 9.33 x 10^-42. On the log plot it falls on a straight line from n=5 onward, and a linear fit of log(error) vs n gives a slope of -0.962425. The theoretical value log(1/phi^2) is -0.962424. The two agree to six significant figures.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/78_convergence.png)

Other starting pairs land in the same place. Lucas numbers (2, 1) reach the float-precision limit of phi at the same rate. A random pair like (850, 637) does the same. Scaling the entire Fibonacci sequence by 10 leaves the ratios unchanged, which is obvious but worth checking once.

A small table:

| seeds | ratio at n = 50 (float) | converged? |
|---|---|---|
| 1, 1 (Fibonacci) | 1.6180339887 | yes |
| 2, 1 (Lucas) | 1.6180339887 | yes |
| 850, 637 (random) | 1.6180339887 | yes |
| 511, 270 (random) | 1.6180339887 | yes |

The perturbed-recurrence checks worked too. Tribonacci ratios head to 1.83929 (the tribonacci constant), nowhere near phi. The degenerate recurrence x_{n+1} = 2 x_n - x_{n-1} gives ratios that creep toward 1 from above, also unrelated to phi. So the phi limit is specifically tied to the F_{n+1} = F_n + F_{n-1} structure.

The second figure is the famous tiling: squares with sides 1, 1, 2, 3, 5, 8, 13, 21 nested into a rectangle, with a logarithmic spiral r = a exp(b theta) overlaid (where b = ln(phi) / (pi/2), so the spiral quadruples its radius every quarter turn at the golden rate). The squares and the spiral were drawn separately; the fact that they fit together at all is what makes the picture look like a single object.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/78_spiral.png)

There was no surprise in the result, but two small things did stand out. First, the convergence is much faster than I had remembered. By n = 12 (F_13/F_12 = 233/144 = 1.6180555...) the ratio is already correct to four decimal places. Kepler, working by hand, would have hit the right neighborhood by the time he wrote down a dozen terms. Second, the agreement between the empirical slope and log(1/phi^2) is so tight (1.000000 to 4 digits) because the geometric rate is essentially exact: there are no higher-order corrections, just the single exponential factor (psi/phi)^n on the residual.

## What it may mean

This is mathematics rather than physics, so "may mean" is the wrong tense. What the exercise does illustrate is the general pattern that the long-run behavior of a linear recurrence is dictated by the dominant eigenvalue of its companion matrix, with the subdominant eigenvalue setting the convergence rate. Anywhere this kind of additive growth shows up (population dynamics with two age classes, certain coupled oscillators, simple Markov-chain mixing rates), the same phi-or-not-phi question becomes a question about the polynomial whose roots are the eigenvalues. For F_n that polynomial is x^2 - x - 1, whose larger root is phi by construction.

It also clarifies what the "golden ratio in nature" claim actually buys you. The result applies cleanly to systems that obey, or approximately obey, the Fibonacci recurrence: sunflower seed packing on a logarithmic spiral, pine-cone scales, the branching counts in phyllotaxis. Past those cases, claims about Nautilus shells, the Parthenon, the human body, and assorted aesthetic objects rest on weaker evidence; many of the "phi appears here" findings in the popular literature have not survived careful measurement (see George Markowsky's 1992 paper *Misconceptions about the Golden Ratio*). The math is precise, but the cultural penumbra around it is not.

The one piece I would still like to see is a controlled phyllotaxis dataset where the divergence angles between successive primordia are measured on hundreds of real plants. If the predicted angle 360 / phi^2 ≈ 137.5 degrees is a population mean, the distribution and its spread are the more interesting object.

## Loose ends

A few caveats worth flagging. The "random pair" starting conditions still have a generic component along the dominant eigenvector; if you carefully picked the initial pair (a, b) = (1, -1/phi) you would zero out the phi^n term and the ratio would not converge to phi (it would converge to psi instead). Such initial conditions form a measure-zero set, so they never come up by accident, but they exist. Float arithmetic saturates fast, and without exact rational error tracking the log plot looks like noise from n=40 onward; the Decimal step is necessary to see the geometric tail. With another afternoon I would pull a published phyllotaxis dataset and check whether the divergence-angle distribution centers on 137.5 degrees and how tight the spread is.
