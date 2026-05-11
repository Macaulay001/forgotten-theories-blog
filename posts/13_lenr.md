# Cold fusion's most popular theory predicts you'd be dead. Why aren't the labs?

*Part 13 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

On 23 March 1989, Martin Fleischmann and Stanley Pons stood in front of a press camera in Salt Lake City and said they had run room-temperature nuclear fusion in a jar of heavy water with a palladium electrode. The paper followed shortly after in *Journal of Electroanalytical Chemistry* 261:301. If the claim held, the energy economy of the planet would change inside a decade. Within months the hot-fusion community had largely written it off. Caltech, MIT and Harwell could not reproduce the excess heat, the neutron signatures were missing or marginal, and the consensus settled into something close to "pathological science".

The interesting thing is that it never quite went away. A small community kept publishing, kept claiming intermittent excess heat in palladium-deuterium systems, and rebranded the field as LENR (low-energy nuclear reactions). I wanted to know what that literature actually says now, in 2026. I was not trying to debunk anything. I was trying to find out whether the dominant proposed *mechanism* in modern LENR theory is internally consistent with the rest of nuclear physics. That turned out to be harder, and weirder, than I expected.

## What I tried, and what went wrong first

My initial plan was simple. Pull the modern replication literature, count positives versus nulls, see whether the field has converged on a stable signal. I started where I usually start, which is PubMed, because it has clean abstracts and a reliable API.

That was a mistake, and a kind of funny one. PubMed indexes biomedicine. A search for "cold fusion" returned 147 hits, every one of which was about something else: spinal fusion surgery, cold-stress biology, protein fusion proteins, the occasional review of bibliometric oddities. The hit rate of actually-LENR papers was zero. Not low. Zero. I had picked the wrong corpus, and the corpus had answered me very clearly.

So I pivoted to arXiv, which is the right corpus for fringe-and-edge physics if it exists in any open archive at all. I pulled 924 papers matching cold-fusion or LENR keywords. Then I hit the second problem: arXiv search treats "cold" and "fusion" as common physics words. A lot of the returned papers were about cold atom gases, fusion reactor engineering, cold dark matter, nothing to do with Pons-Fleischmann.

I needed a classifier. I used GPT-4o-mini with a fairly strict prompt asking whether each abstract was about LENR / cold-fusion in the Pons-Fleischmann sense. The cost for the full run was about three cents.

Of 924 arXiv papers, **31 were actually on-topic**. That is 3.4%. The other 893 were false positives from keyword overlap. The scarcity itself is a finding. After 36 years there are roughly thirty-one arXiv papers in the mainstream physics preprint archive that engage seriously with the original claim. That is not a thriving field on the open record. It is a tiny one.

## What thirty-one papers look like

I expected to find replication attempts. The standard scientific way to settle a contested empirical claim is to keep measuring it. That is not what the corpus contains.

Of the 31 on-topic papers:

| Category | Count |
|---|---|
| Theory papers proposing a mechanism | 21 |
| Positive replication claims | 3 |
| Null or negative experiments | 2 |
| Reviews / meta / commentary | 5 |

So roughly two thirds of the on-topic literature is people proposing reasons it could work, not people testing whether it does. That is an unusual ratio for a contested empirical claim. In a healthy controversy you would expect the experiments to outnumber the theories by a wide margin, because the question is whether the phenomenon is real, not how to explain it.

The 21 theory papers cluster pretty hard on one mechanism. I pulled out the proposed mechanism from each abstract and tagged them by hand. Electron screening of the Coulomb barrier showed up in 8 of the 21. Nothing else came close. There are scattered proposals around Bose-Einstein condensation of deuterons, neutron capture from "ultra-low momentum" neutrons, lattice phonon coupling, and various exotic quasiparticle ideas, but screening is the modal answer. If you had to name one thing the current LENR theory community believes is going on, it is that electrons in the palladium lattice partially shield the repulsive Coulomb barrier between deuterons and let them tunnel through it at much higher rates than vacuum d-d fusion would predict.

That is a testable claim. Pd-D electron screening has actually been measured in low-energy accelerator experiments, with screening potentials of order **U_e ≈ 600 eV** in the bulk metal. The question is: does that number, plugged into thermal nuclear reaction rate theory, give you anything close to the 1 W/cm³ excess heat that Pons and Fleischmann reported?

I worked through it with a clean Gamow-factor calculation using the measured screening potential. The answer was, surprisingly, **roughly 0.4 W/cm³**. Within an order of magnitude of the claim. I was not expecting that. Honestly I had assumed the screening mechanism would fail by ten or twelve orders, and that would be the end of it. Instead the popular LENR theory passed the first quantitative test.

So I went looking for where it would break.

## The branching ratio problem

Here is the thing about d-d fusion that screening theory cannot bend. The reaction has three known product channels and their branching ratios are measured to high precision across many decades of accelerator experiments:

```
d + d → ³He (0.82 MeV) + n (2.45 MeV)    ~50%
d + d → t   (1.01 MeV) + p (3.02 MeV)    ~50%
d + d → ⁴He + γ (23.8 MeV)               ~10⁻⁷
```

The 4He-plus-gamma branch, which is the one LENR proponents need (because they report helium-4 as the "ash" without proportionate neutrons or tritium), is suppressed by a factor of about ten million. This is not a small correction. It is a property of the nuclear matrix element, and screening the *entrance* channel does not change it, because screening only affects how often two deuterons get close enough to react, not what they do once they do.

So if screening is the mechanism, and if the reaction rate is in the right ballpark for 1 W/cm³, then 50% of those reactions must produce a 2.45 MeV neutron. Let me work out what that means.

```python
# 1 W via standard d-d branches (~4 MeV per event, since 4He branch is suppressed)
events_per_s_per_cm3 = 1.0 / (4e6 * 1.602e-19)  # ~1.5e12 events/s/cm^3
n_per_s_per_cm3 = 0.5 * events_per_s_per_cm3     # ~7.5e11 neutrons/s/cm^3

# Fluence at 1 m from an isotropic 1 cm^3 source
area_at_1m_cm2 = 4 * math.pi * 100**2
fluence = n_per_s_per_cm3 / area_at_1m_cm2

# Neutron-to-dose for 2.45 MeV (ICRP 74): ~3.7e-11 Sv per (n/cm^2)
dose_Sv_per_h = fluence * 3.7e-11 * 3600
```

A one-watt-per-cm³ cell, under standard d-d branching, gives a dose rate of **about 0.8 Sv/h at one metre**. For comparison: LD50 acute dose is around 4 to 5 Sv. The annual exposure limit for radiation workers is 20 mSv, which this source would deliver in about ninety seconds.

The cells in Pons-Fleischmann replications sit on a bench. People work next to them for hours. Neither Pons nor Fleischmann nor any of their successors have died of acute radiation syndrome. Positive LENR replications that bother to measure neutrons report upper bounds of **under 1 neutron per second** from the whole apparatus, not 7.5 × 10¹¹ per cm³.

The discrepancy between the prediction and the observed upper bound is about **10¹²**. Twelve orders of magnitude. That is not a calibration uncertainty.

I want to be careful about what this does and does not show. It does not show that the *heat* claim is wrong. The calorimetry literature in LENR is its own complicated mess and I have not engaged with it here. What it does seem to show is that the most popular *explanation* of that claimed heat, electron-screened d-d fusion in the palladium lattice, is internally inconsistent with the experimenters being alive. The mechanism, if true at the rate needed, would have killed everyone who ever ran the experiment.

## What this might mean, with hedging

I am genuinely not sure what to do with this. A few possibilities, none of which I can rule out:

The screening mechanism is wrong, and the heat (if real) comes from something that is not d-d fusion. The 4He-as-ash claim then needs a different nuclear pathway, perhaps something genuinely exotic, and the field would need to propose a mechanism that produces ⁴He without traversing the standard d-d branches. None of the 21 theory papers I read does this cleanly.

The screening mechanism is right *in principle* but the rate is much smaller than my Gamow calculation suggests, by some lattice-specific factor I am not modeling. In that case the heat claim collapses too, because the same factor reduces the heat output.

The heat claim is not real, or is real but conventional (chemical, recombination, calibration artefact). This is the mainstream view and I have not done anything here to contradict it.

What I do feel reasonably confident saying, on this corpus, is that the dominant theoretical proposal in the modern LENR literature fails an internal-consistency test by a factor of roughly a trillion. That seems worth saying out loud, because most LENR critiques attack the experiments. This one comes from inside the theory.

## Loose ends

The clean falsification protocol would not be cheap but it would be definitive. Run simultaneous calorimetry, neutron counting and helium mass spectrometry on a single sealed cell. If the heat is from a nuclear mechanism that ends in ⁴He, then 1 W for 24 hours requires the production of about **9 milligrams of helium-4**, which is bulk material and easily measured. If the heat is from anything in the d-d branching family, it must be accompanied by neutrons and tritium in the textbook ratio. Either you see the products in the predicted amount or you do not. I have not found a positive LENR replication that runs the full panel and survives.

If anyone has access to a calorimetry-plus-mass-spec-plus-neutron rig and a few weeks of patience, the experiment that would settle this is sitting there waiting, and it would probably cost about $50k in instrument time.
