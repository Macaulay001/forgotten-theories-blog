# Tesla's Earth wireless-power scheme delivers 3 picowatts to your phone. The numbers are unkind.

*Part 14 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Nikola Tesla spent the winter of 1899 in Colorado Springs running a coil the size of a barn. His notebooks from that season, published much later as the *Colorado Springs Notes*, describe lightning arcs of fantastic length and a conviction that the Earth itself was a resonator. By 1901 he was raising Wardenclyffe Tower on Long Island, a 187-foot wooden structure topped with a copper dome, financed by J.P. Morgan, and meant to deliver electric power to anywhere on Earth without wires. The money ran out, the tower was dismantled in 1917, and Tesla's reputation drifted toward mysticism. His central technical claim, though, is testable: that the cavity between the Earth's surface and the conducting ionosphere resonates near 8 Hz and could be excited with enough quality that a single ground transmitter would light lamps in Bombay.

The 8 Hz number is real. Winfried Schumann predicted the cavity modes in 1952 and they were measured a decade later. The fundamental sits at 7.83 Hz. That part of Tesla's intuition was correct. The rest is what I wanted to check.

## What I tried

The physics is forgiving here. You do not need a full electromagnetic solver to ask whether the scheme makes sense. The Earth-ionosphere shell behaves like a leaky spherical waveguide. You can write down the energy stored in a uniform azimuthal magnetic field filling the shell, divide by the cavity Q to get the loss rate, and that loss rate is the power your transmitter has to supply to hold the field steady.

The cavity volume is roughly the surface area of Earth times an effective height of 80 km, which gives about 4 × 10¹⁹ m³. Big. The observed quality factor of the Schumann fundamental is the part that hurts. Decay-time measurements of natural Schumann transients give Q ≈ 4 to 6. I used 5. Tesla seems to have assumed something one to two orders of magnitude better, which is the move that makes his arithmetic work.

For the receiver side I used a tuned loop antenna with its own Q of about 10, a square loop of area A, induced EMF V = 2πfBA, matched to a load. Standard physics. The script lives at `src/tesla_calc.py`. Here is the part that does the work:

```python
def required_power_for_field(B_target, f, h, Q):
    # Energy stored in the cavity, then divided by Q per radian
    V = (4/3) * math.pi * ((R_E + h)**3 - R_E**3)
    U = (B_target**2 / (2 * mu0)) * V
    P = U * 2 * math.pi * f / Q
    return P

# Tesla's tower: invert for the field 10 MW can sustain
B2 = P_T * 2 * mu0 * Q_typical / (V * f1 * 2 * math.pi)
B = math.sqrt(B2)
```

Then I asked two questions. First, what does Tesla's claimed 10 MW Wardenclyffe transmitter actually buy you globally? Second, what would it take to deliver a field strong enough to do anything useful, say a hundred microtesla, the kind of thing you can rectify with a reasonable antenna?

## What happened

The 10 MW number gives an azimuthal B-field of about 0.25 nanotesla, averaged over the cavity. For comparison, natural Schumann amplitudes already sit around 1 nT. Tesla's tower would have nudged the background up by a quarter, no more.

The receiver side is where the picture turns bleak. A 1 m² tuned loop in a 0.25 nT, 7.83 Hz field collects about **3 × 10⁻¹² watts**. Three picowatts. A modern smartphone charges at roughly 5 W. The gap is about 10⁹, a factor of a billion. Even a 100 m² loop antenna, the size of a small house, gets to 3 nanowatts. Still nine orders of magnitude shy of doing anything.

You can run the inversion the other way. Suppose we want a useful 100 µT field globally, a field strength where a modest coil rectifier produces meaningful power. The required radiated power comes out to roughly 10²² W. The total solar flux intercepted by Earth is about 1.7 × 10¹⁷ W. The Tesla scheme, scaled to deliver real power, demands a transmitter five orders of magnitude brighter than the Sun as seen from Earth.

| Target B-field | Power to sustain | Power at 1 m² loop |
|---|---|---|
| 1 nT (Schumann amplitude) | 1.6 × 10⁶ W | 5 × 10⁻¹¹ W |
| 1 µT | 1.6 × 10¹² W | 5 × 10⁻⁵ W |
| 100 µT (useful) | 1.6 × 10¹⁶ W | 0.5 W |
| Tesla's 10 MW achieves | 0.25 nT | 3 × 10⁻¹² W |

The 100 µT row is the one that almost works at the receiver, half a watt into a square meter, but the transmitter has to push 16 petawatts to do it. Global electricity demand is around 3 TW. You would need to build something five thousand times the size of every power plant on Earth, then dump that energy into a cavity whose Q is 5, meaning about 80% of what you radiate is lost per cycle.

The Q is the whole story. Tesla's intuition came from the resonant transformer, where you can store thousands of times the input energy by carefully matching frequencies. The Schumann cavity does resonate, but it is a leaky resonator. The ionosphere is not a perfect conductor, the ground is not a perfect conductor, the air in between has nonzero conductivity at ELF, and lightning strikes keep injecting noise at every frequency. The cavity loses energy to all of these channels. You cannot raise Q by being clever about the transmitter design, because Q is a property of the medium, not the source.

There is a subtler issue. The mode shapes of the Schumann fundamental are zonal harmonics, broad sinusoidal patterns wrapping the planet. To couple efficiently you want a vertical electric dipole much taller than 187 feet. Tesla seemed to know this and wanted to keep building bigger, which is part of why Wardenclyffe ballooned in cost. But scaling the antenna does not change the cavity Q. It only changes how efficiently you pour power into a leaky bucket.

I should be careful here. The numbers above use a uniform-field approximation. The real cavity has spatial structure. A receiver placed at a field antinode could do a little better, maybe a factor of two or three. A receiver at a node, near the geomagnetic equator for certain modes, does worse. Neither moves the answer by nine orders of magnitude.

## What it may mean

The Schumann resonance is one of the loveliest pieces of geophysics. The cavity is real, it does ring, you can hear it in any quiet ELF antenna setup, and the modes match theory to a fraction of a hertz. Tesla was right that something resonant lives between the ground and the sky. He appears to have been wrong about how lossy that resonance is, and the loss is what kills the scheme.

This suggests, tentatively, that any global wireless power approach built on Earth-cavity excitation runs into a hard physics wall at the Q of the medium. You could imagine working around it with higher modes, the 14.3 Hz second harmonic, the 20.8 Hz third, but the observed Q does not improve much at higher modes (some reports put it slightly higher, others slightly lower, none get past about 8). You could imagine phased arrays of transmitters distributed across continents, but that improves coupling efficiency, not cavity Q.

Modern resonant wireless power, the kind in your phone's Qi charger or in MIT's Witricity demonstrations, works at MHz frequencies and centimeter to meter ranges. It avoids the cavity-loss problem by not relying on a cavity. The fields fall off fast, which is the price of high efficiency. Tesla's dream of global broadcast trades range for nearly total loss.

I do not want to claim more than the arithmetic supports. The calculation here is a back-of-envelope with two free parameters, the cavity height and the cavity Q, both of which are observationally constrained to better than a factor of two. The conclusion sits on numbers that move by, at most, a factor of a few. The gap between 3 picowatts and 5 watts is nine orders of magnitude. Factor-of-two uncertainties do not close that gap.

## Loose ends

There are pieces I have not modeled. The day-night ionosphere asymmetry could in principle let you set up a traveling wave with slightly different losses. The auroral electrojet adds structure I ignored. Solar cycle variation shifts the D-layer height and the Q by 10 to 20%. None of these look like they could rescue the scheme, but a more careful treatment is worth doing if anyone has a specific mode geometry in mind. The script and the JSON output live in `14_tesla_earth_resonance/`. Replicating the calculation takes about a minute on a laptop.
