# Richardson said Britain's coast has no length. I tested his exponent on four Koch curves and a circle.

*Part 25 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Lewis Fry Richardson was a Quaker, a pacifist, and a meteorologist who spent the 1940s and 50s trying to find numerical patterns in the causes of war. One detail kept tripping him up. Different sources gave different lengths for the same border. The Spain-Portugal frontier was 987 km in one almanac and 1214 km in another. The Belgian-Dutch border was off by 20%. He decided this was not an error but a signal, and in a posthumously published 1961 paper called "The problem of contiguity" he laid out the reason. If you measure a wiggly boundary with a ruler of length s, the total length you get is approximately

    L(s) = K s^(1 - D)

with D some number greater than one, characteristic of the boundary. Shrink the ruler, the measured length grows. For the west coast of Britain Richardson found D close to 1.25. For the smoother coast of South Africa, D was close to 1.02. Mandelbrot picked this paper up six years later and gave D its modern name, fractal dimension, in his 1967 Science note "How long is the coast of Britain?".

The paradox is famous enough now that it sits in undergraduate textbooks. What I wanted to know was simpler. If you sit down with a clean synthetic curve whose theoretical D is known exactly, does the divider method Richardson described actually recover it? And does the recovered exponent distinguish a rough coast from a smooth one in a way that a working scientist could trust?

## What I tried

The cleanest test piece is a generalised Koch curve. Start with a unit segment. Replace it with four shorter segments forming a bump of apex angle theta. Repeat n times. The resulting curve has a theoretical dimension

    D = log(4) / log(2 + 2 cos theta)

For the standard Koch snowflake (theta = 60°) you get D ≈ 1.2619, almost exactly Richardson's value for Britain. Smaller theta gives a flatter bump and a smaller D. Larger theta gives a more violent zigzag and a larger D. I built four curves: theta = 30°, 45°, 60°, 75°, each at recursion level 6, which gives 4096 segments and a smallest feature size around 1.4 × 10⁻³ in unit length. As a control I generated a 6000 point circle, which should give D exactly 1.

Then I implemented the divider method, the same one a 1920s cartographer would have used with a pair of compasses. You set the compass to opening s, plant one leg on the start of the curve, swing the other forward until it crosses the curve, mark that crossing, lift, plant, repeat. The number of steps times s is the measured length. I picked s values logarithmically spaced from about 6 × 10⁻³ up to about 2 × 10⁻¹, which is the range where the ruler is well above the smallest curve feature but well below the full extent of the curve. That gives the "inertial range" where you expect a clean power law.

The exponent comes out of a straight line fit of log L versus log s. The slope is (1 - D), so D = 1 - slope. I bootstrapped the residuals 500 times to get a 95% confidence interval, which catches fit noise but not geometry noise.

## What happened

The circle came in at D̂ = 1.006 with a tight CI of [1.003, 1.010]. Good. The method has not invented roughness where there is none. Now the Koch curves:

| Curve | theta | D theory | D hat | 95% CI |
|-------|------:|---------:|------:|--------|
| smooth | 30° | 1.053 | 1.043 | [1.036, 1.049] |
| middling | 45° | 1.129 | 1.089 | [1.077, 1.099] |
| Britain-like | 60° | 1.262 | 1.130 | [1.120, 1.141] |
| rough | 75° | 1.501 | 1.256 | [1.230, 1.282] |

A few things to notice. The recovered D̂ values are monotone in theta, with no overlap between adjacent confidence intervals. The method distinguishes the four curves cleanly. But D̂ is systematically below the theoretical value, and the gap widens as the true D grows. For the smoothest case the bias is about 0.01. For the Britain-like case it is 0.13. For the roughest case it is 0.25.

This is not new. Klinkenberg's 1994 review of fractal estimators flagged exactly this: the divider method underestimates D for prefractals of finite recursion depth, because the curve runs out of fine structure below a certain scale and the ruler eventually just measures the polygonal hull. With only 6 levels of recursion, the inertial range is narrow, and the fitted slope gets dragged toward zero by the saturation tail.

Here is the core of the divider walk, lifted from the script.

```python
def ruler_length(pts, s):
    cur = pts[0].copy()
    total, i = 0.0, 0
    while True:
        j = i + 1
        while j < len(pts) and np.linalg.norm(pts[j] - cur) < s:
            j += 1
        if j >= len(pts): break
        a, b = pts[j - 1], pts[j]
        # interpolate to find point exactly s away from cur
        d, e = b - a, a - cur
        A, B, C = d @ d, 2 * d @ e, e @ e - s * s
        t = (-B + np.sqrt(B * B - 4 * A * C)) / (2 * A)
        cur = a + t * d
        total += s
        i = j - 1
    return total
```

It is the kind of code Richardson would have written by hand on graph paper, basically a forward search with linear interpolation between vertices. No magic.

I also did a scale-split sanity check on the Britain-like case. Half the s values low, half high, fit each separately. D̂(small s) = 1.155, D̂(large s) = 1.144. Close enough that the slope is roughly stable across the fitted decade and not just an end-effect.

So the headline result on this sample is two-sided. Yes, Richardson's law fits. The log-log plot is straight to the eye across a decade. The order of D values lines up with intuitive roughness. No, the absolute number does not match theory without correction. If someone hands you a coastline shapefile and asks for its fractal dimension, you should report it with at least a tenth-of-a-unit uncertainty bar, and you should note that the divider method tends to under-report.

## What it may mean

Richardson's paradox stopped being a paradox the moment people accepted that "length" is not the right summary statistic for a wiggly object. It is the same move that made it acceptable to summarise a distribution by more than its mean. A coastline has a length-versus-resolution function, parameterised by an exponent. That exponent is the thing that travels across measurement methods.

The interesting modern echo is that this same exponent shows up in biology with much higher stakes than border disputes. Lung bronchial branching has a measured D between 1.1 and 1.2 across mammals. Neuron dendrites cluster around 1.4. The vascular tree of the retina sits near 1.7 by area-perimeter estimates, and changes detectably in diabetic retinopathy. In every one of these systems, the exponent is doing work that no single length number could do, and it is being estimated with the same methods Richardson sketched in his war-causes paper.

The result on this sample also says something about how to read published fractal-dimension numbers in ecology and geomorphology. If someone reports a coastline D of 1.25, that may be from a divider-method fit on a finite-resolution shapefile. The true exponent, if such a thing even exists for a real coast, could plausibly be 0.1 or 0.2 higher than the reported number. Comparing two coasts measured at the same resolution with the same method is fine. Comparing one paper's D to another paper's D is not, unless you know they used the same s range and the same depth of zoom.

## Loose ends

I did not pull real coastline data. Natural Earth's 1:10m shapefile is available and I would have liked to feed Britain into the same pipeline to see whether D̂ ≈ 1.13 there too, matching the Britain-like Koch. The recursion-depth bias correction (Tricot 1988) was also skipped; with a depth-aware fit you can usually claw back about half of the missing exponent. And the divider method is one of three classical estimators. Box-counting and area-perimeter would give independent numbers, and the spread across the three is itself a better uncertainty bar than the bootstrap CI I quoted. A weekend of extra work would close all three gaps.
