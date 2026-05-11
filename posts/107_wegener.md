# I rotated South America onto Africa with Bullard's 1965 Euler pole. The mean misfit was 565 km.

*Part 107 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In January 1912 Alfred Wegener gave a lecture in Frankfurt arguing that the continents had once been joined. The case was old in spirit and young in evidence. Francis Bacon had noticed the Atlantic coastline match in 1620. Antonio Snider-Pellegrini had drawn a Pangaea-like map in 1858. What Wegener did differently was assemble four classes of evidence at once: the geometric fit of the South American east coast against the African west coast, fossils of Mesosaurus and Glossopteris on continents now thousands of kilometres apart, geological strata that lined up across the Atlantic when you slid the continents back together, and Permian glacial deposits in South America, Africa, India, and Australia that only made sense if those landmasses had once huddled near the pole.

The book came in 1915. Mainstream geology, particularly the English-speaking branch of it, did not buy it. The 1926 American Association of Petroleum Geologists symposium in New York is the famous flashpoint. The objection was mechanism: how on Earth could a continent plough through the ocean floor. Harold Jeffreys's calculations in 1924 showed that the forces Wegener had invoked, tidal friction and centrifugal "Polflucht" forces, were several orders of magnitude too weak. Drift died in respectable circles for about fifty years and was revived in the 1960s when paleomagnetism showed apparent-polar-wander curves diverging between continents and Hess and Vine-Matthews supplied the missing engine (seafloor spreading).

I wanted to know what the geometric argument actually looks like as a number. Wegener never had a computer. Edward Bullard, who would later run the famous 1965 fit, did not have one in 1912 either. I had one for a morning.

## What I tried

The plan was to take a coastline for the east of South America, a coastline for the west of Africa, apply the Bullard 1965 rigid Euler rotation that the geophysics community has been quoting for sixty years, and measure how close the two curves come on a sphere.

I traced both coastlines by hand from world-atlas memory at roughly five-degree intervals: Cabo Orange to Tierra del Fuego on the South American side, Cape Spartel to Cape Agulhas on the African side. Each set of waypoints was then densified by linear interpolation to 200 points. That is much coarser than the digitised 500-fathom isobath Bullard used, which is one reason my numbers come out larger than his, but it preserves the salient shape: the bulge of Brazil at Recife, the indent of the Gulf of Guinea, the long southward tapers of both coasts.

The rotation itself is one Rodrigues formula away from elementary. You take each coastline point (lon, lat), convert to a unit vector on the sphere, rotate the vector about the Bullard pole at 47°N, 32°W by an angle (positive 57°, which closes the Atlantic), and convert back. The Bullard pole and angle are the canonical values from Bullard, Everett and Smith's 1965 paper in the Royal Society's Philosophical Transactions, where they were the first to find them by numerical optimisation.

To get a misfit number I computed, for every rotated South American point, the great-circle distance to its nearest neighbour on the African coast, and then the same thing the other way (Africa to rotated SA), and averaged the two. This is the simplest defensible curve-to-curve metric on a sphere.

The control test was the part I cared most about. If the coastline match were a Rorschach blot, I should be able to rotate Australia, or North America's east coast, onto Africa and get a similar number with a different Euler pole. So I ran the same machinery for both, with a much wider search range so that each control gets every chance to find its best fit.

The third piece was a small simulation of apparent polar wander, the diagnostic that revived drift in the 1950s. If two continents are riding around independently, the paleomagnetic pole positions they "see" in their rocks at successive ages trace different curves through latitude and longitude space. If they are part of the same rigid Earth, they trace the same curve. I did not run real paleomagnetic data, just a sketch of the topology.

## What happened

The Bullard rotation closes the South Atlantic almost as well as you would expect from looking at any school globe.

![Wegener fit](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/107_wegener_fit.png)

Numbers across the four fits, in mean nearest-neighbour misfit on the sphere:

| pairing | best-rotation mean misfit |
|---|---|
| SA east on Africa west, Bullard pole | 565 km |
| SA east on Africa west, local optimum | 445 km |
| North America east on Africa west, best | 717 km |
| Australia west on Africa west, best | 1,277 km |

The unrotated raw misfit between SA and Africa is about 4,800 km, roughly the present width of the Atlantic at 20°S. Bullard knocks that down by a factor of nine in a single rigid rotation. A local search around the Bullard pole shaves a further 120 km off; not a lot, given how coarse my coastlines are.

The controls are the part that should be persuasive. North America's east coast is roughly the same length as Africa's west coast and bows in roughly the same direction; with 60 degrees of freedom in pole longitude and latitude and angle, the optimiser can find a rotation that brings the mean misfit down to ~700 km. That is the closest a wrong continent gets. Australia, much shorter and oriented wrong, never gets below ~1,300 km no matter where you put the pole.

Bullard's own paper, using digitised continental shelves at the 500-fathom contour rather than hand-traced surface coastlines, reported a mean misfit of about 88 km. My 565 km is what you should expect from coastlines sampled every five degrees and aligned at the modern shoreline rather than at the shelf break. The ratio between the SA-Africa fit and the controls is what survives, and it survives clearly.

Here is the core of the rotation, the part that does the actual closing of the Atlantic. No imports, no boilerplate:

```python
axis = lonlat_to_xyz(pole_lon, pole_lat)        # unit vector at Euler pole
axis = axis / np.linalg.norm(axis)
theta = np.deg2rad(angle_deg)
dot = np.einsum("...i,i->...", xyz, axis)[..., None]
cross = np.cross(axis, xyz)
# Rodrigues' rotation formula on the sphere
rotated = xyz * np.cos(theta) + cross * np.sin(theta) + axis * dot * (1 - np.cos(theta))
lon, lat = xyz_to_lonlat(rotated)
misfit_km = mean_nearest_neighbour_distance(rotated_coast, fixed_coast)
```

The synthetic APW panel shows the other half of the story. With drift switched off, two continents' simulated paleomagnetic poles cluster within about 2.4 degrees of one another over a 200-million-year window. With each continent given its own independent Euler rotation, the curves diverge by about 11 degrees on average. The Runcorn and Irving measurements in the 1950s saw real APW curves diverging by tens of degrees between Europe and North America. That is the observation that made the geometric coincidence start looking like a geometric necessity.

## What it may mean

Three thoughts.

The geometric fit alone is, as far as I can tell from this exercise, hard to explain by chance once you have a metric. The probability of finding a rigid rotation that closes a 4,800 km gap to ~500 km, when the next-best continental pairing cannot do better than 700 km despite a comparably long candidate coastline, is the kind of fact that should have given mid-century geology more pause. Jeffreys was correct in 1924 that nobody had a mechanism. He was not correct that the geometric evidence was negligible.

The second is a methodological point about how long a wrong consensus can hold. Wegener died in Greenland in 1930 with his theory still mocked. The textbook chronology now puts the rehabilitation around 1965, with Bullard's fit and the Vine-Matthews paper on magnetic seafloor stripes. That is a 35-year gap during which the evidence was largely there and the framework to interpret it was not. It is the cleanest case I can think of where a single decisive instrument, the seafloor magnetometer, broke a logjam that decades of argument over coastlines had failed to break.

The third is small. Bullard's pole at 47°N, 32°W is not arbitrary. It sits where it sits because the Atlantic spreading ridge has a particular geometry, and any rigid rotation that closes the ocean has to be perpendicular to the spreading direction. The fit and the mechanism, treated as separate problems for fifty years, turn out to be the same problem under different lighting.

## Loose ends

The biggest caveat is that my coastlines are hand-traced at five-degree resolution rather than digitised from a Natural Earth shapefile at sub-degree resolution. With higher-resolution data the SA-Africa misfit should drop toward Bullard's published 88 km and the controls should worsen relative to it. The synthetic APW panel is illustrative, not a refit of real Runcorn-era pole positions. If anyone wants to push this further, swapping in real shelf-edge contours and the GPMDB paleomagnetic database would take about a day.
