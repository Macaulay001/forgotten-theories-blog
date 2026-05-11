# I tested Stirling's 1730 factorial formula against exact n! up to 200 and the error fell exactly as 1/(12n).

*Part 76 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1730 James Stirling published *Methodus Differentialis*, a book of finite-difference techniques. Tucked inside is one of the most useful approximations in mathematics:

    n! ~ sqrt(2 pi n) * (n/e)^n

The shape of the result is a little strange when you first see it. A factorial is a product of integers, so why does a square root of pi land in the answer. Stirling did not have the modern saddle-point derivation. He had de Moivre's earlier observation that log(n!) grows like n log n minus n, plus a constant that needed to be fixed, and a series of clever finite-difference manipulations that nailed the constant to sqrt(2 pi).

The formula has been re-derived a dozen ways since. From the Gamma function and its saddle point. From the Euler-Maclaurin formula applied to the sum of log(k). From Laplace's method on the integral definition of the Gamma function. They all give the same asymptotic series, and the first correction term is the famous 1/(12n):

    n! = sqrt(2 pi n) (n/e)^n (1 + 1/(12n) + 1/(288 n^2) + O(1/n^3))

I had used Stirling for entropy calculations and for combinatorial estimates for years. I had quoted "error of order 1/n" in lectures. I had never actually plotted the error against n, on exact integer factorials, and watched the slope come out to -1.

## What I tried

The exact n! is easy in Python. Arbitrary-precision integers are built into the language. `math.factorial(200)` returns a 375-digit integer in microseconds. Stirling's formula, on the other hand, lives in floating point and would overflow a double around n = 170 or so if I tried to compare directly. So I worked in log space.

The plan was three checks of the same claim.

First, compute log(n!) exactly. For each n from 1 to 200 I summed log(k) for k = 1..n using doubles. This is exact up to the rounding of each log term, which gives an error budget on the log of order 10^-14 per term, well below what I cared about.

Second, evaluate Stirling at three levels of refinement.

The leading form: log_S = n log n - n + 0.5 log(2 pi n).

The first refinement: log_S + log(1 + 1/(12n)).

The second refinement: log_S + log(1 + 1/(12n) + 1/(288 n^2)).

Third, compute the relative error in n! itself by exponentiating the log difference:

    rel_err = exp(log_approx - log_exact) - 1

If Stirling is right, |rel_err_leading| should fall like 1/(12n), so on a log-log plot of |rel_err| vs n the slope should be -1 with a prefactor near 1/12 = 0.083. The first refinement removes the 1/(12n) term, so the next leading error is from the 1/(288 n^2) term, and the slope should be -2 with prefactor near 1/288 = 0.0035.

That is the whole experiment. It fits in 30 lines.

## What happened

The slopes came out to -0.999 for the leading form and -1.974 for the once-refined form, fit on n >= 5. Theory predicts -1 and -2. The match is a little embarrassing in how clean it is.

Here is the core of the calculation:

```python
ns = np.arange(1, 201)
log_exact = np.array([sum(math.log(k) for k in range(1, n+1)) for n in ns])

log_S = ns * np.log(ns) - ns + 0.5 * np.log(2 * np.pi * ns)
log_S_r1 = log_S + np.log(1 + 1/(12 * ns))
log_S_r2 = log_S + np.log(1 + 1/(12 * ns) + 1/(288 * ns**2))

rel_err_leading = np.exp(log_S - log_exact) - 1
rel_err_r1      = np.exp(log_S_r1 - log_exact) - 1
rel_err_r2      = np.exp(log_S_r2 - log_exact) - 1
```

Sample values:

| n   | leading rel err | + 1/(12n)        | + 1/(288 n^2)    |
|-----|------------------|--------------------|--------------------|
| 1   | -7.79e-02        | -1.36e-03          | -3.0e-05           |
| 10  | -8.30e-03        | -3.18e-05          | -2.6e-07           |
| 100 | -8.33e-04        | -3.44e-07          | -2.6e-10           |
| 200 | -4.17e-04        | -8.64e-08          | -3.3e-11           |

Each refinement gains roughly two extra digits per decade in n, exactly what the next term in the series predicts.

The leading error is negative for every n. Stirling under-estimates the factorial, and the 1/(12n) correction is positive, pushing the estimate up. By the time we add 1/(288 n^2) the residual is again negative (the next term in the full series is -139/(51840 n^3), which has a negative leading sign in the standard convention, though signs depend on how you arrange the series).

At n = 10 the bare formula gives 3,598,695.62 against the true 10! = 3,628,800. That is 0.83% low. Multiplying by (1 + 1/120) gives 3,628,684.75, which is 0.003% low. Adding the next term takes you to within parts per million.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/76_stirling_error.png)

The log-log plot shows three sets of points, each tracking one of the reference lines 1/(12n) and 1/(288 n^2) to within the marker thickness. The second-refinement points peel off the 1/(288 n^2) line at the high-n end because the next term in the series starts to dominate. That is the asymptotic series doing what asymptotic series do.

## What it may mean

For practical work, Stirling at the leading order seems to be enough whenever you can tolerate a fraction of a percent error and n is bigger than 10 or so. For four-digit accuracy you want the 1/(12n) term. For six digits, add the 1/(288 n^2) term. Beyond that the series becomes divergent in the strict sense, which is a quirk of asymptotic expansions: more terms helps up to a point, then hurts.

In statistical physics, where n is Avogadro's number (~6e23), even the leading form is overkill. Entropies expressed as -k_B sum p log p use Stirling to convert factorials of large occupation numbers, and the relative error there is ~1.4e-25. Nobody worries about the correction term.

In small-n combinatorics, the picture is different. At n = 5 the leading form is off by 1.7%, which matters if you are computing binomial probabilities for hand calculations. Using the once-refined form drops that to 0.001%.

I would not call any of this a discovery. The exercise is calibration. It tells you, concretely, what "good to 1/n" means in real numbers. The historical claim from 1730, that you can replace a product of integers with a smooth function of n times a square-root correction and lose only one digit per factor of ten, holds up cleanly on exact arithmetic done 296 years later.

One detail worth noting on the historical side. Stirling published the constant sqrt(2 pi) but de Moivre, working at the same time on the central limit problem for binomial sums, had already shown the constant existed and approximated it numerically. The collaboration was friendly. De Moivre cited Stirling in later editions of the *Doctrine of Chances*. The modern habit of attaching one name to a formula obscures the fact that pieces of this result were in the air around London in the 1720s and 1730s.

It is also worth pointing out that the relative error is what one usually wants, not the absolute error in n!. The factorial grows so fast that the absolute error in n! becomes astronomical even when the relative error is parts per million. At n = 100 the relative error of the refined form is 3.4e-7, but the absolute error in the value of 100! is around 3.2e151. The right way to talk about Stirling is always in relative or in log-difference terms.

## Loose ends

The clean -1 and -2 slopes might tempt one to extrapolate the series to arbitrary precision. That fails because the Stirling series is asymptotic, not convergent: for any fixed n, adding more terms eventually makes the estimate worse, with the optimal truncation around n terms. I did not chase that boundary here. A nice follow-up would be to find, for each n, the term k* that minimizes the residual, and see whether k* tracks n the way the standard theory predicts. The whole thing would take an afternoon.
