# Coulomb's 1785 inverse-square law holds at 18th-century precision, and modern null tests now pin the exponent to within 3 parts in 10^16

*Part 64 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Charles-Augustin de Coulomb published his torsion-balance results in
1785. He hung a fine silver wire from the top of a glass cylinder,
attached a horizontal straw with a charged pith ball at one end, and
brought a second charged ball up to a known distance. The wire twisted.
Coulomb read the angle, related it back to a force by Hooke's law for
the wire, and reported that the electrostatic repulsion between two
small spheres fell off as the inverse square of their separation. The
exponent he could write down was 2, within a few percent.

The interesting thing is that two earlier people had already half-
proved it without telling many readers. Joseph Priestley argued in 1767
from a tossed-off Newtonian analogy: if electricity inside a hollow
charged conductor produces no force, then by the same logic as Newton's
shell theorem, the force law has to be exactly inverse square. Henry
Cavendish ran a sealed-sphere null experiment in 1772 and bounded the
exponent to about 2 plus or minus 0.02. He did not publish. Coulomb got
the credit, fairly, because he showed his numbers.

I wanted to know two things. First, can you actually recover n = 2 from
a Coulomb-grade dataset, meaning five distances with about 1% noise?
And second, how tight is the modern bound, and what does that bound say
about photon mass?

## What I tried

The setup is a numerical caricature of Coulomb's torsion balance. Two
identical charges of 1 nC. Five separations chosen to span a factor of
ten: 2, 4, 8, 12, and 20 cm. At each separation the "true" force is
k q1 q2 / r^2 with k = 8.988 x 10^9 N m^2 / C^2. I then added 1%
Gaussian multiplicative noise per measurement, averaged four repeats,
and asked the data to give back the exponent without telling it what
the right answer was. Five points is not many. Coulomb in his published
paper does not show many more. The geometry was designed to look like
something an 18th-century experimentalist could actually defend.

The fit is OLS in log-log. If F = k_eff / r^n then log F = log k_eff -
n log r, so the slope of log F against log r is minus n. The exponent
falls out of a single linear regression, and the uncertainty falls out
of the residual variance. No fancy machinery.

A second thing I wanted to check is superposition, the property that
the force from two source charges on a test charge equals the vector
sum of the two pairwise Coulomb forces. Coulomb himself does not test
this explicitly in his 1785 paper; it appears to be taken on the
authority of Newton-style mechanics. So I simulated two geometries: two
sources on a line with the test charge at the origin, and two sources
at right angles. The "measured" net force is a noisy realization of the
true vector sum. If superposition holds, the measurement and the
vector-sum prediction agree within the noise.

Here is the heart of it, the function that turns the data into n:

```python
def fit_exponent(r, F):
    lx = np.log(r)
    ly = np.log(F)
    slope, intercept = np.polyfit(lx, ly, 1)
    n_fit = -slope
    k_fit = np.exp(intercept)
    resid = ly - (slope * lx + intercept)
    s2 = np.sum(resid**2) / (len(r) - 2)
    slope_se = np.sqrt(s2 / np.sum((lx - lx.mean())**2))
    return n_fit, k_fit, slope_se
```

Six lines of arithmetic. Coulomb did this by reading angles off a
graduated arc and rearranging algebra by hand. The math is the same.

## What happened

The fit comes back clean. With seed = 1785, the recovered exponent is
n = 1.997 with a standard error of 0.003, giving a 95% confidence
interval of [1.991, 2.002]. The interval covers 2 comfortably. The
fitted prefactor k_eff (which carries the q1 q2 product) matches the
true k_eff within about 0.7%. That is essentially what Coulomb claimed
from his 1785 data. Roughly two centuries of precision later, the
qualitative answer has not moved.

![Coulomb log-log fit and residuals](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/64_coulomb_loglog.png)

The log-log panel shows the five points lying on the fitted line, with
the textbook n = 2 dashed line passing through them at the same slope.
The residuals panel is more interesting. The deviations are tiny and
they sit symmetrically around zero, with no monotonic trend. This is
what you want a noise-only residual to look like. If a Yukawa
screening term were leaking into the law (a force that decays faster
than 1/r^2 at large r), the residuals would curve downward at large r.
They do not, at this noise floor.

Superposition holds too. The in-line geometry gives a predicted net
force of 4.49 x 10^-6 N; the noisy "measurement" sits within 1.3% of
that, well within the per-component 1% noise. The perpendicular
geometry agrees to 0.4%. Neither result is a stringent test, but
neither shows any structural deviation from vector addition.

| Geometry | predicted F (N) | rel. diff |
|---|---|---|
| In-line, two sources at 5 cm and 10 cm | 4.49 x 10^-6 | 1.3% |
| Perpendicular, sources at 5 cm on x and y | 5.08 x 10^-6 | 0.4% |

The five-point exponent is the headline, but it is also where the
distance to the modern bound becomes startling. My fit says |n - 2| is
under 1%. Williams, Faller, and Hill ran a 1971 experiment that put
|n - 2| under 2.7 x 10^-16. That is fourteen orders of magnitude better
than my synthesized torsion balance, and it is roughly the same factor
of improvement over Cavendish 1772 as Cavendish was over a naive
eyeball reading. The Williams-Faller-Hill apparatus is a nested set of
hollow conductors. If the inverse-square law is exact, the field inside
the inner conductor when a voltage is applied to the outer one is
identically zero. Any non-zero reading is an upper bound on the
deviation.

A common way to interpret a deviation from 1/r^2 is to write the
potential as a Yukawa: V(r) ~ q exp(-r / Lambda) / r. Then |n - 2| over
a lab scale r_lab is related to the Compton wavelength Lambda of a
hypothetical massive photon. The photon mass satisfies m_gamma c^2 ~
hbar c / Lambda. Plugging in the WFH bound and r_lab ~ 0.5 m gives
m_gamma c^2 < ~10^-22 eV, roughly. That is the photon weighing less
than a part in 10^39 of an electron. Later experiments using satellite
magnetometers and pulsar timing have pushed the bound further still,
into the 10^-18 to 10^-27 eV range depending on what assumptions you
allow. My own synthesized lab can only manage ~3 x 10^-9 eV, which is
to say that 1% torsion-balance noise leaves a lot of headroom that
modern techniques have closed.

## What it may mean

Coulomb's law appears to be one of the tightest empirical statements
in physics. The 2.7 x 10^-16 figure is not a casual bound. It is
roughly four orders of magnitude tighter than tests of Newton's
gravitational inverse-square at the same kind of distance. The reason
is that electromagnetic forces are huge compared to gravity, so a tiny
fractional deviation produces a measurable signal where the
gravitational version would produce nothing.

The practical consequence is that anything you build on top of
F = k q1 q2 / r^2, which is most of classical electromagnetism, sits
on a load-bearing wall that has been stress-tested to a precision few
physical laws ever reach. Gauss's law, the existence of conductors as
true equipotentials, the shell theorem for spheres, the photon being
massless within current bounds: all hang on the same exponent.

That said, the 1971 number is not the last word. If anyone ever finds
|n - 2| above the current limit, the implications run deep: a photon
mass, possible extra spatial dimensions at sub-millimeter scales, and
revisions to QED gauge structure. Nobody has, on this sample.

## Loose ends

Five points and synthetic noise is a thin reconstruction. A more
honest re-test would use an actual torsion balance, calibrated against
a known electric field, and would include the parasitic forces Coulomb
glossed over (image charges on nearby walls, residual gravity,
electrostatic drift on the wire). I also stretched the Yukawa-to-
photon-mass conversion to one significant figure; the original WFH
paper uses a slightly different lever arm and a more careful
derivation. With another week I would dig up the WFH preprint and
re-derive the bound at their actual apparatus geometry, then compare
to the more recent dark-photon limits from satellite magnetometry.
