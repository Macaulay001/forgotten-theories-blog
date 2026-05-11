# I redid Lavoisier's 1774 calcination mass balance on ten metals with modern NIST atomic weights. Phlogiston still can't survive it.

*Part 101 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1703 Georg Ernst Stahl, professor of medicine at Halle, published *Specimen Becherianum*, an extension of Johann Becher's earlier ideas about the "earths" hidden inside burning bodies. Stahl's central claim was clean. Every flammable substance contains a principle, *phlogiston*, that escapes during burning. Wood is phlogiston-rich; ash is what remains when the phlogiston has gone. Metals are phlogiston-rich too. Heat them in air and they shed phlogiston, ending up as their "calx", what we would now call an oxide. Heat the calx with charcoal, a phlogiston donor, and you get the metal back.

For most of the 18th century this was chemistry. Cavendish, Priestley, Scheele, and Black all worked in phlogiston's vocabulary. The Encyclopédie article on combustion in 1753 is built around it. Then between roughly 1774 and 1789 it disappeared from the curriculum, replaced by Lavoisier's oxygen theory, and now it appears in chemistry textbooks only as a one-paragraph cautionary tale about the bad old days. That is the part that always nagged me. A framework that organised a generation of careful experimentalists doesn't usually collapse in fifteen years unless the killing blow is unusually sharp. I wanted to feel the sharpness.

## What I tried

The killing blow, as standard histories tell it, was the mass balance. If phlogiston is a real substance leaving the body, then a metal that has been calcined should weigh *less* than the metal you started with. Robert Boyle had already noticed in the 1670s that tin gained weight when calcined in a sealed vessel, but he attributed it to "fire particles" passing through the glass. Lavoisier closed the loop in 1774 by weighing the sealed vessel itself before and after, finding no change in total mass, then opening it and finding that the air inside had lost weight equal to the gain in the metal.

That is one experiment on one metal. I wanted ten, with modern numbers, so the pattern would be unmistakable. The plan was small and arithmetic. Hand-encode NIST molar masses for ten common metals (Mg, Al, Ca, Fe, Cu, Zn, Sn, Pb, Hg, Ag) and the standard binary oxide each forms when heated in air. Compute the mass change per mole of metal. Compare against the two competing predictions:

- Phlogiston: Δm < 0 (mass is lost as phlogiston escapes).
- Oxygen: Δm = n_O · M(O), positive and equal to the mass of oxygen incorporated.

Then run the more interesting test. In 1772 Louis-Bernard Guyton de Morveau tried to save phlogiston by giving it *negative weight*. The proposal sounds absurd today but it was a serious move at the time: if the loss of phlogiston could make a body heavier, the mass-balance objection went away. The question is whether one consistent molar mass for phlogiston can fit every metal. If it can, the rescue is at least internally coherent. If the required value drifts metal by metal, you've just added a free parameter per experiment, and the theory has stopped being a theory.

The atomic-mass data is from NIST. The oxide stoichiometries are the textbook room-temperature products (FeO/Fe2O3 ambiguity aside; I picked the most common air-stable oxide). Nothing here needed more than a calculator. The point was the spread.

## What happened

Every metal in the table gained mass. The smallest gain was silver at +7.4%, the largest was aluminium at +88.9%. The mean was +33.7%. Across ten metals, the phlogiston prediction (Δm < 0) is correct in zero cases. The oxygen prediction is correct in ten.

```python
M_O = 15.999
for sym, M_metal, oxide, M_oxide, n_metal, n_O in METALS:
    dm_per_mol_metal = (M_oxide - n_metal * M_metal) / n_metal
    pct = dm_per_mol_metal / M_metal * 100.0
    # Guyton 1772 rescue: m(metal) - M_phlog = m(calx)
    # so M_phlog must be -dm_per_mol_metal
    M_phlog_required = -dm_per_mol_metal
    print(sym, f"{pct:+.1f}%", f"M_phlog={M_phlog_required:+.1f} g/mol")
```

A slice of the output:

| metal | oxide  | mass change per mole metal | % gain |
|-------|--------|----------------------------|--------|
| Mg    | MgO    | +16.0 g | +65.8 |
| Al    | Al2O3  | +24.0 g | +88.9 |
| Fe    | Fe2O3  | +24.0 g | +43.0 |
| Cu    | CuO    | +16.0 g | +25.2 |
| Sn    | SnO2   | +32.0 g | +27.0 |
| Hg    | HgO    | +16.0 g |  +8.0 |
| Ag    | Ag2O   |  +8.0 g |  +7.4 |

![Every metal gains mass; Guyton's required negative phlogiston mass is not constant](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/101_mass_balance.png)

Now the Guyton de Morveau rescue. To make m(metal) - M_phlog = m(calx) work per mole of metal, you need M_phlog to be exactly the negative of each Δm. The values come out at -8.0 (Ag), -16.0 (Mg, Ca, Cu, Zn, Pb, Hg), -24.0 (Al, Fe), -32.0 (Sn). The mean is -18.4 g/mol with a standard deviation of 6.2 g/mol, a coefficient of variation of 0.34. If phlogiston is one substance, this should be one number, not four.

The pattern is the giveaway. Each required value is an integer multiple of 8.0. The multiplier is exactly (number of oxygen atoms per metal atom in the oxide). Plot the required mass against that ratio and every point lands on the line y = 16.0 · x. Phlogiston, in the modernised Guyton picture, has to behave exactly like oxygen running backwards. That isn't a rescue of phlogiston. It's a renaming of oxygen with the sign flipped on the books.

The other rescue that 18th-century chemists tried, "the air absorbs phlogiston and the metal absorbs fire-matter", is observationally indistinguishable from the negative-weight version once you only measure masses. To distinguish them you need to track *volume* of air, which is what Lavoisier finally did in his sealed-retort experiments with mercury: heating Hg in a measured air volume produces red HgO and shrinks the air by about one-fifth, and the lost air can later be recovered by heating the HgO. Phlogiston theory has no spare degree of freedom to account for that volume change without postulating yet another substance.

## What it may mean

Stahl's framework was not stupid. It correctly grouped combustion, calcination, and reduction as instances of one process, and it correctly predicted that you could move "something" between bodies during these reactions. The error was the sign of that something's mass. Once careful balances became standard, the framework had only one move left: assign negative weight to the carrier. That move fails as soon as you ask it to be consistent across substances.

What I take from the exercise is narrower than "the old chemists were wrong". The forgotten part of phlogiston is that it wasn't defeated by a single dramatic experiment. It was defeated by a tabulation across many substances that any 1770s chemist with a good balance could have done, and several did. The reason it took eighty years was partly inertia and partly that mass-only data is genuinely consistent with a negative-weight rescue if you only look at one metal at a time. The cross-substance variation is what kills it.

This is not an argument by analogy to anything modern. It's a reminder that single-experiment confirmations of theories with a free parameter mean less than they feel like they mean.

## Loose ends

I used the dominant binary oxide for each metal at standard temperature and pressure, which papers over genuine ambiguity for iron (FeO vs Fe2O3 vs Fe3O4) and tin (SnO vs SnO2). Picking the other oxide for these two changes percentages by roughly a third but doesn't change a single sign. The historical 1770s analytical balance had a precision near 0.01 g on samples of a few grams, which is two orders of magnitude looser than the NIST molar masses I used, so the historical data was already enough. If anyone has Guyton's own 1772 numbers digitised, I'd like to overlay them; my searches turned up the title page of *Digressions Académiques* but not the tables. Replication of the arithmetic above takes about ten minutes.
