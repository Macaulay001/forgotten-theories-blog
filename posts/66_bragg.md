# I simulated Bragg's 1913 X-ray law for NaCl, copper, and iron. The peak angles agreed to a hundredth of a degree.

*Part 66 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In November 1912, Max von Laue published the first X-ray diffraction photograph of zinc sulfide. The image showed a regular pattern of spots around the direct beam. Nobody quite knew what to do with it. The wave nature of X-rays was still being argued about, and the regular atomic spacing of crystals was a conjecture rather than a measurement. Within four months, a 22-year-old physics graduate in Cambridge named William Lawrence Bragg had the explanation, and within a year he and his father had used it to determine the atomic arrangement of rock salt. The younger Bragg got the 1915 Nobel Prize at 25, an age record that still stands.

The relation he wrote down,

  n * lambda = 2 * d * sin(theta),

is one of the most-photographed equations in physics. Every textbook draws the two parallel planes, the incoming and outgoing rays, the little path-difference triangle that yields the factor 2*d*sin(theta). The equation looks almost too simple for the work it does. So I wanted to check it the way you check a tool you depend on. Pick three crystals whose lattice constants are known to four figures, plug the geometry in, and see whether the predicted angles for Cu K-alpha radiation match the values on the official PDF cards that diffractometer software ships with.

## What I tried

The plan was a five-line simulator and three textbook materials.

A cubic crystal has plane spacings d_hkl = a / sqrt(h^2 + k^2 + l^2) where (h, k, l) are Miller indices. For each integer triple up to (5, 5, 5), I computed d, then the Bragg angle theta from sin(theta) = lambda / (2*d). Cu K-alpha has lambda = 1.5406 Angstrom, which is the most common laboratory X-ray source. That gives 2*theta in degrees, which is what a diffractometer reports.

Geometry alone is not enough. Most (hkl) values produce zero diffracted intensity because the atoms inside the unit cell scatter out of phase with each other. The structure factor takes care of this. For a unit cell with atoms at fractional positions (x_j, y_j, z_j) and atomic numbers Z_j,

  F_hkl = sum_j Z_j * exp(2*pi*i * (h*x_j + k*y_j + l*z_j))

The intensity is |F_hkl|^2, weighted by a Lorentz-polarisation correction that accounts for how a powder sample distributes orientations and how polarised the scattered X-rays are. For face-centred cubic (FCC) crystals like copper, the four-atom basis at (0,0,0), (1/2,1/2,0), (1/2,0,1/2), (0,1/2,1/2) makes |F|^2 vanish unless h, k, l are all even or all odd. For body-centred cubic (BCC) iron, the two-atom basis at (0,0,0) and (1/2,1/2,1/2) makes |F|^2 vanish unless h+k+l is even. NaCl is a two-sublattice FCC with Na at (0,0,0) and Cl at (1/2,1/2,1/2), and the contrast in Z between sodium (11) and chlorine (17) creates a characteristic alternation between strong "all-even" peaks and weaker "all-odd" peaks.

I used lattice constants a = 5.640 Angstrom for NaCl, 3.615 for Cu, and 2.866 for alpha-Fe. The implementation fits in a screen.

```python
# core loop: enumerate (hkl), compute Bragg angle and structure factor
for h, k, l in product(range(hkl_max + 1), repeat=3):
    if h == k == l == 0:
        continue
    d = a / np.sqrt(h*h + k*k + l*l)
    s = lam / (2.0 * d)
    if s >= 1.0:
        continue                     # angle would exceed 90 deg
    two_theta = np.degrees(2.0 * np.arcsin(s))
    F = sum(Z * np.exp(2j*np.pi*(h*x + k*y + l*z))
            for Z, (x, y, z) in basis)
    rows.append((h, k, l, d, two_theta, abs(F)**2))
```

For each row I also recomputed lambda from the predicted 2*theta and d to check that the relation closes. That residual is the tightest test of the geometric law.

## What happened

The Bragg residual, max |lambda - 2*d*sin(theta)|, came out at 4.4 x 10^-16 Angstrom for NaCl and 2.2 x 10^-16 for Cu and Fe. That is machine precision in double-precision floating point. There is no slack in the relation. The geometry is exact, and the only error is in the arithmetic.

The selection rules killed the right peaks. For copper, 63 of 71 candidate reflections in my (hkl) range had |F|^2 below 10^-6 (in units where the allowed peaks are O(100)), and the 8 survivors were exactly the all-even-or-all-odd combinations. For iron, 25 of 31 candidates went to zero, leaving only h+k+l even. NaCl gave a richer pattern with 16 allowed peaks out of 169 candidates.

The big test was whether the angles matched what a working crystallographer would expect. They did.

| material | (hkl) | predicted 2*theta | tabulated 2*theta (ICDD) |
|----------|-------|------------------:|-------------------------:|
| Cu       | (111) | 43.32 deg         | 43.30 deg                |
| Cu       | (200) | 50.45 deg         | 50.43 deg                |
| Cu       | (220) | 74.13 deg         | 74.13 deg                |
| Fe       | (110) | 44.68 deg         | 44.67 deg                |
| Fe       | (200) | 65.03 deg         | 65.02 deg                |
| NaCl     | (200) | 31.70 deg         | 31.70 deg                |
| NaCl     | (220) | 45.45 deg         | 45.45 deg                |

Two decimal places of agreement, on hand-typed lattice constants and the K-alpha_1 wavelength. The residuals against the PDF cards are about 0.01-0.02 degrees, which is roughly the precision at which the lattice constants themselves are published. If I had used a five-figure value for a, the agreement would tighten.

![Simulated powder diffraction patterns for NaCl, Cu, and Fe at Cu K-alpha. Top three peaks of each pattern are labeled with their Miller indices. The selection rules are visible: FCC Cu shows only (111), (200), (220), ...; BCC Fe shows only (110), (200), (211), ...](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/66_diffraction_patterns.png)

One detail surprised me a little. In the NaCl pattern, the (111) reflection (all odd) is much weaker than (200) (all even), because in (111) the Na and Cl sublattices interfere destructively: F goes as Z_Cl - Z_Na = 6, whereas in (200) it goes as Z_Cl + Z_Na = 28. The intensity ratio is roughly (6/28)^2 ~ 0.046. The experimental ratio reported in the ICDD card 05-0628 is ~0.10 (allowing for multiplicities and form factors). Same order, same sign, off by a factor of two in absolute intensity. That gap is exactly what you would expect when you neglect the angle-dependent atomic form factor f(sin theta / lambda), which I did. The geometric law is exact; the intensity model is not.

## What it may mean

What strikes me, doing the exercise, is how literally Bragg's two-parallel-planes argument maps onto a real crystal. The relation does not contain a fitted parameter. It contains lambda, which is set by the X-ray tube, and d, which is set by the lattice. The fact that an integer multiple of the wavelength has to fit into 2*d*sin(theta) is just a path-difference accounting, and the accounting holds at sub-femtometre residuals for every (hkl) family I tried.

The 1913 paper effectively said: if you accept that X-rays are waves and that crystals are periodic stacks of atoms, the diffraction angles are forced. Within months that prediction was being used to read out the atomic arrangements of solids one by one. NaCl was first, then KCl, then diamond, then graphite. By the 1950s the same method was being pushed to resolve protein structures, and the geometry of the technique had not changed. The reciprocal lattice was a more efficient bookkeeping device, but the underlying argument was still Bragg's.

It is rare for a relation this simple to remain this exact a century after its proposal. The closest analogue may be the lens equation in optics. Both are geometric, both are linear, both work because the underlying wave physics happens to be very clean in the relevant regime.

## Loose ends

With another week I would do two things. The first is add atomic form factors f(sin theta / lambda) from the international tables and a Debye-Waller factor for thermal vibration, and check whether the predicted intensity ratios match measured powder patterns to within 10 percent. The second is to run the same simulator on a non-cubic crystal, where d_hkl has a more interesting formula that depends on all six lattice parameters, and see whether the prediction still tracks tabulated angles. If anyone has a clean experimental powder pattern of a hexagonal or monoclinic material and is willing to share it, I will fit it for the cost of an evening.
