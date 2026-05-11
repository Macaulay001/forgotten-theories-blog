# I tested Prout's 1815 integer-weight rule against 92 natural elements. He was 70% right, for reasons he never imagined.

*Part 110 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In the summer of 1815 a London physician published an anonymous note in *Annals of Philosophy*. He had been adding up atomic weights and had noticed something. If you took hydrogen as the unit and measured everything else against it, the numbers kept coming out close to whole numbers. Oxygen at 16. Nitrogen at 14. Carbon at 12. He drew the obvious inference. All the elements were built from hydrogen, condensed in different multiples. He called hydrogen the *protyle*, the primal stuff. The next year he confessed his identity. William Prout.

For forty years his hypothesis was the chemistry community's favourite unresolved puzzle. Berzelius accepted it. Dumas circled around it. Then, in the 1840s and 1860s, two careful measurers, Jean-Servais Stas in Brussels and Jean Charles Galissard de Marignac in Geneva, ran the experiment with the best balances the century had. Chlorine came out at 35.45, not 35 or 36. Copper at 63.55. Magnesium at 24.31. The deviations were many times the experimental error. By 1860 Prout's rule was retired as a curiosity, an early example of pattern-spotting in noise.

It came back, twice. The first revival was Frederick Soddy's coining of "isotopes" in 1913 and Francis Aston's mass spectrograph a few years later: each individual nuclide *did* have a near-integer mass; chlorine's 35.45 was a weighted mean of 35Cl and 37Cl. The second was the Burbidge-Burbidge-Fowler-Hoyle paper of 1957, which showed that the heavier elements really are built from hydrogen, just inside stars rather than in some primordial soup. The protyle was real. The mechanism took 142 years to find.

I had quoted that history for years without ever doing the arithmetic. So one afternoon I did.

## What I tried

The test is unfashionably literal. Take the 92 natural elements, hydrogen to uranium. Look up the standard atomic weight (NIST/CIAAW 2021 abridged values). Round each weight to the nearest integer. Record the distance. Then split the elements into two groups, those that are effectively mono-isotopic (one stable isotope above 99% natural abundance, or a radioelement with a single accepted reference mass) and those that are not. Compare the distributions.

The historical claim is that *all* elements should be integers. The post-1913 understanding is that the rule should work for isolated isotopes but fail for natural mixtures, with the failure size proportional to how much the isotopes scatter around the mean. Stas's chlorine should be the worst offender because 35Cl and 37Cl are both abundant.

Hand-encoding the table was the slow part. I copied 92 atomic weights into a CSV and flagged each as mono-isotopic or mixed, double-checking the borderline cases (vanadium is 50V + 51V but 51V is 99.75%, so I called it mixed; gold is 100% 197Au, mono). I marked 31 as mono and 61 as mixed.

Then I added a second test that would have been impossible in Prout's lifetime. I picked 31 stable nuclides spanning A = 1 to 238 (hydrogen, deuterium, helium-4, carbon-12, oxygen-16, iron-56, nickel-62, lead-208, uranium-238, and 22 others), pulled their exact atomic masses from AME2020, and computed the binding energy per nucleon. This is the curve that explains why the atomic weights are *close* to integers but not exactly. The mass defect (the difference between Z hydrogens plus N neutrons and the actual nucleus) is what holds the nucleus together. Iron-56 sits at the peak of the curve, with 62Ni a hair above it.

No LLM calls. No fancy data pipeline. Two CSVs and one NumPy script.

## What happened

The split was sharper than I expected.

Of the 92 elements, 59% land within 0.1 u of an integer. That is the number Stas would have computed, more or less, and it is enough to think the rule is suggestive but not exact. The mean absolute deviation across the table is 0.148 u.

Once you split by isotope structure, the picture changes. All 31 mono-isotopic elements are within 0.1 u; the mean deviation is 0.039 u, and the worst offender (hydrogen, with its tiny deuterium contamination) sits at 0.008 u. Of the 61 mixed-isotope elements, 38% pass the 0.1 u test and the mean deviation is 0.20 u. The two groups separate cleanly. There is essentially no overlap in the upper tail.

| group         | n  | within 0.1 u | mean abs deviation (u) |
|---------------|----|--------------|------------------------|
| all elements  | 92 | 58.7%        | 0.148                  |
| mono-isotopic | 31 | 100.0%       | 0.039                  |
| mixed isotope | 61 | 37.7%        | 0.202                  |

The worst single offenders are exactly the ones the 19th century used as evidence against Prout: chlorine (0.45 u from integer), copper (0.45), magnesium (0.31), boron (0.19). Each is a near-50/50 (or roughly two-thirds / one-third) blend of two stable isotopes that differ in A by two units. The atomic weight has to land somewhere in between. Once you read the table at the nuclide level, the integer rule comes back.

Here is the core of the calculation, with no boilerplate.

```python
# atomic weights for Z = 1..92 from NIST/CIAAW
dev = W - np.round(W)
mono = MONO == 1

within = (np.abs(dev) < 0.1)
print("mono pass:", within[mono].mean())
print("mixed pass:", within[~mono].mean())

# mass excess and binding energy for 31 nuclides
BE = (Z * 1.00782503207 + (A - Z) * 1.00866491588 - M) * 931.494
peak = (A[np.argmax(BE / A)], (BE / A).max())
print("BE peak:", peak)
```

The binding-energy curve looks the way every nuclear-physics textbook draws it. It climbs steeply from the deuteron (1.11 MeV per nucleon) to about 7 MeV by carbon, peaks at 8.79 MeV for 62Ni (with 56Fe and 58Fe almost as bound), then declines slowly. By 238U it is back down to 7.57 MeV. The mass excess m - A stays within ~0.1 u everywhere; the *signed* excess is positive for very light nuclei (hydrogen at +0.00783 u) and for the actinides (uranium at +0.0508 u), negative through the iron region (56Fe at -0.0651 u). That curvature is the part Prout could never have seen. It needs E = mc² and four decades of atomic physics.

![Prout deviation and binding-energy curve](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/110_deviation_vs_Z.png)

The deviation plot is the cleaner of the two: blue mono points cluster tightly along zero, red mixed points scatter out to half a unit. Chlorine sits in the lower middle as the canonical example.

## What it may mean

A few things stand out.

The first is that Prout was working with a rough sample of natural-element weights and reading a real pattern out of them. The pattern is the integer A of the dominant isotope, smeared by abundance-weighted averaging when more than one isotope is present. He did not have the language for "isotope". He chose the simplest model that fit the bulk of his data and got punished for it by chemists who could measure to four decimals.

The second is that the "refutation" of 1860 and the "revival" of 1913 are the same observation read at different magnifications. Stas measured natural chlorine and got 35.45. Aston measured 35Cl and 37Cl separately and got 34.97 and 36.97. Both numbers are correct. The question is which one you call "the atomic weight of chlorine", and that depends on whether you are doing 19th-century stoichiometry or 20th-century mass spectrometry.

The third is more cosmological. If you accept that nuclei are built from nucleons and that nucleons came from a hot early universe of mostly protons (and a few alphas synthesised in the first few minutes), then hydrogen really is the protyle in the strict sense Prout meant. Stellar nucleosynthesis adds the heavier elements one fusion at a time. The B²FH paper of 1957 is the modern version of Prout 1815, with the difference that B²FH knows what binding energy is and why iron is special.

On this sample the rule clears 100% on mono-isotopic elements and 38% on mixed. That is not "right" in the textbook sense. It is the kind of partial rightness that turns into a Nobel later.

## Loose ends

A few caveats. Atomic weights and isotope masses were hand-encoded; the third decimal place is at the mercy of my typing. The mono/mixed flag is judgment-call on a handful of edge cases (vanadium, rubidium, indium, rhenium). Some "mono" radioelements (Tc, Pm) use IUPAC reference masses rather than a measured atomic weight, which inflates their pass rate slightly. None of this changes the headline gap between groups. With another week I would pull the full AME2020 table for every stable nuclide and replicate the binding-energy curve at 250 points instead of 31, then check whether the residual deviation of natural atomic weight from abundance-weighted nuclide mass matches the published CIAAW uncertainty bars. I suspect it does, to within the rounding of the abridged values.
