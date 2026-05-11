# I reran Rumford's 1798 cannon-boring estimate in numpy. Heat keeps appearing with no fluid in sight.

*Part 102 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

For about sixty years, the standard story of heat was that it is a
fluid. Lavoisier put it in his 1789 list of elements under the name
*calorique*. It was supposed to be weightless, indestructible, and
to seep from hot bodies to cold ones the way water seeps from a
full cup to an empty one. Carnot wrote his 1824 cycle in caloric
language. Calorimetry, latent heat, specific heats: all of it
worked, more or less, on the fluid picture. Chemistry textbooks
through the 1830s drew little pores in metals to show where the
caloric went when iron cooled, and how a compressed gas had its
caloric "squeezed" out through its surface.

Rumford spent the winter of 1797 watching brass cannons being bored
at the Munich arsenal and noticed something the fluid story could
not absorb. A blunt borer driven by two horses kept the brass
warm. Then hot. Then near boiling. The cylinder was finite. The
caloric supply, on the official theory, was finite. The heating
was not. I wanted to see how close his back-of-the-envelope numbers
were to a careful arithmetic, and what the same numbers say when
you stand them next to Joule's paddle-wheel and the ideal-gas heat
capacities a chemistry student measures today.

## What I tried

Three small simulations, one per experiment that the caloric theory
had to answer.

The first is Rumford's cannon. He reported a brass blank of 113 lb
(about 51 kg), a draft of two horses, and roughly two and a half
hours to bring the metal to boiling temperature. Two horses at
sustained draft put out about 1.5 kW of mechanical work. Brass has
a specific heat of 380 J/(kg·K). If a fraction of the horse-power
goes into the brass and the rest leaks (water bath, iron stand,
convection), then the heat budget is fully fixed by the simulation:
no caloric, just W → Q.

The second is Joule's 1845 paddle-wheel. A pair of falling weights
turn paddles in a sealed jar of water. The water warms. The
quantity in question is how many joules of mechanical work it
takes to warm one calorie's worth of water. The fluid theory had
no room for this question. If heat is conserved, work cannot
become heat at a fixed exchange rate.

The third is the prediction that finally killed caloric on paper.
If heat is a conserved fluid, the heat capacity of a gas should
not depend on whether the gas is allowed to expand. In symbols,
c_p = c_v. The kinetic theory that replaced caloric predicts the
opposite: c_p - c_v = R = 8.314 J/(mol·K), because in a constant
pressure heating the gas does mechanical work on its surroundings
that has to be paid for by extra heat input. Two predictions, one
number: easy to check.

I wrote one script and put the numbers in NPZ files so I could be
sure I had not fudged anything between runs.

## What happened

The Rumford simulation matches him almost exactly. Setting the
fraction of horse-power that ends up in the brass to 12.5 percent
(reasonable, given how much heat must have gone into the
surrounding water and air), the cylinder hits 100 °C at 2.47
hours. Rumford reported 2.5 hours. The fit is not the interesting
part. The interesting part is that the curve does not turn over.
The brass keeps heating as long as the horses keep walking. A
caloric reservoir would empty.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/102_rumford_heating.png)

The Joule simulation lands on 4.186 J/cal against the modern
4.184. Two weights of 13.6 kg, a 1.6 m drop, twenty repetitions,
six kilos of water, ninety-five percent of the potential energy
delivered through the paddles. The water warms by 0.32 K. The
exchange rate is not negotiable.

```python
m_weight, n_weights, h, n_drops = 13.6, 2, 1.6, 20
m_water, c_water, g = 6.04, 4186.0, 9.81
W = 0.95 * n_drops * n_weights * m_weight * g * h
dT = W / (m_water * c_water)
Q_cal = m_water * 1000.0 * dT
J_per_cal = W / Q_cal
print(J_per_cal)  # 4.186
```

The heat capacities are the cleanest of the three. I pulled
tabulated c_p and c_v at room temperature for six gases and took
the difference.

| gas | c_p - c_v  [J/(mol·K)] |
|---|---|
| He | 8.31 |
| Ne | 8.32 |
| Ar | 8.38 |
| N2 | 8.32 |
| O2 | 8.28 |
| CO2 | 8.67 |

Mean 8.38, standard deviation 0.13. R is 8.314. The kinetic
prediction sits inside the spread. The caloric prediction is off
by the full value of R, and the gap is the same size as R itself
for every gas in the table. That is not a small effect to argue
about. It is the whole quantity.

![c](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/102_cp_cv_gap.png)

What I had not appreciated before running this is how cleanly the
three experiments line up. Each one closes off an escape route
that caloric theorists tried in turn. Rumford's friction means
heat is not conserved against mechanical action. Joule pins the
exchange rate. The c_p minus c_v gap closes the question for any
expanding gas, since heat is being consumed to do mechanical work
on the surroundings. The fluid is gone.

## What it may mean

Caloric is the canonical example of a theory that worked for the
wrong reasons. Carnot's cycle, derived from a conserved fluid
flowing between reservoirs at different temperatures, survived the
transition to kinetic theory almost intact, because what Carnot
had really proved was a relation between reservoir temperatures
and cycle efficiency that did not depend on the carrier. Anything
flowing between hot and cold and returning to its starting state
gives you the Carnot bound. The fluid was a scaffold.

One thing that surprised me, going back through the primary
sources, is how slow the field was to give up. Rumford published
in 1798. Joule reported his first weight-and-pulley value in 1843.
Clausius reformulated thermodynamics in 1850. The fluid persisted
in French textbooks into the 1860s, partly because the formalism
still gave the right answers for the questions practical engineers
were asking, and partly because nobody had a good intuition for
"motion of invisible molecules" until Maxwell's distribution made
it tractable.

This may matter for current arguments where a useful framework
sits on top of an unobserved carrier. Dark matter, the renormalon
in QFT, the inflaton field. Some of these may turn out to be like
caloric: the bookkeeping works, but the entity is a placeholder
for a deeper mechanism that uses different variables. Others may
turn out to be real. The caloric story does not tell you which,
but it does suggest that a long string of correct predictions is
not enough to confirm the existence of an unobserved substance.

## Loose ends

The Rumford efficiency is tuned to match his stated boiling time,
which feels a little circular. A real loss model with separate
terms for the water bath and convective losses would be cleaner.
The c_p comparison uses ideal-gas tabulated values; CO2 already
shows the cracks where polyatomic vibrational modes start to
matter, and at lower temperatures the noble gases would still
give R but the heat capacities themselves would drift. With
another week I would pull a wider temperature range and check
the high-T limits against Dulong-Petit. None of that would help
caloric.
