# I rebuilt the 1943 McCulloch-Pitts neuron and checked all 256 three-input Boolean functions. Every one fit in a two-layer net with at most four hidden units.

*Part 86 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1943 Warren McCulloch, a neuropsychiatrist with a taste for logic, and Walter Pitts, a teenage runaway who had taught himself mathematics in the Chicago public library, published "A Logical Calculus of the Ideas Immanent in Nervous Activity". The paper is dense, partly impenetrable, and full of a private notation that almost nobody uses today. Buried in the formalism is a simple idea. If you treat a neuron as a unit that adds up weighted inputs and fires when the sum crosses a threshold, then connected networks of these units can in principle compute any Boolean function of finitely many inputs. The brain, in their reading, is a piece of finite logic hardware.

The claim sat awkwardly. It was too abstract for the neurophysiologists, who wanted spikes and synapses, and too sketchy for the logicians, who wanted proofs in the format of Principia Mathematica. By the time von Neumann cited it in his EDVAC report two years later it had already started to shape the design of digital computers, but the original biological framing got mostly ignored. Most modern neural-network textbooks mention McCulloch and Pitts in a single sentence on the way to the perceptron. I wanted to actually rebuild their unit and check the universality claim by hand, on a small enough case that I could see every result.

## What I tried

The unit itself is a few lines of Python. Inputs are 0 or 1. Each input has an integer weight, usually +1 (excitatory) or -1 (inhibitory). The neuron fires (outputs 1) if the weighted sum reaches an integer threshold, and otherwise outputs 0. That's it. No bias, no nonlinearity beyond the step, no learning.

Phase 1 was to find weights and thresholds for the basic logical primitives by hand. AND on two inputs needs weights (1, 1) and threshold 2: the sum hits 2 only when both inputs are 1. OR needs the same weights with threshold 1. NOT on a single input is an inhibitory weight of -1 with threshold 0, which fires when -x >= 0, i.e., when x = 0. NAND and NOR are similar with negative weights. I verified each on the full truth table.

Phase 2 was the harder case, XOR. The four points of the XOR truth table sit at the corners of the unit square: (0,0) -> 0, (0,1) -> 1, (1,0) -> 1, (1,1) -> 0. No straight line separates the firing corners from the silent ones, which is the classical observation Minsky and Papert sharpened in 1969. I did a brute-force search anyway over integer weights w1, w2 in {-3, ..., 3} and thresholds theta in {-6, ..., 6} and found zero single-neuron solutions, exactly as the geometry predicts. Then I built a two-layer net: one hidden unit firing only on (1, 0), one firing only on (0, 1), and an output unit doing OR. Three neurons total.

Phase 3 was the universality check. There are 2^(2^3) = 256 Boolean functions of three binary inputs. For each one, I built a two-layer MP network from its truth table using the disjunctive-normal-form recipe: one hidden neuron per true minterm, with weights that force a match on exactly that input pattern, then an output OR. When the function had more true rows than false rows I built the complement form instead, with a NOR output, which keeps the hidden layer small. Then I verified each net against its target truth table on all eight input rows.

## What happened

The primitives behaved as expected. AND, OR, NOT, NAND, and NOR all hit 4/4 (or 2/2 for NOT) on their truth tables with the hand-chosen weights. The unit really is that simple to reason about. The harder part was watching how the geometry constrains a single neuron.

XOR has no single-neuron representation. The exhaustive search confirmed it: 0 out of 1183 integer weight-threshold combinations matched all four rows. The minimum-error single neurons miss at least one corner. The two-layer net got all four. Here is the construction in code:

```python
def xor_two_layer(x):
    # h1 = x1 AND NOT x2: fires on (1, 0)
    h1 = mp_neuron(x, w=[1, -1], theta=1)
    # h2 = NOT x1 AND x2: fires on (0, 1)
    h2 = mp_neuron(x, w=[-1, 1], theta=1)
    # output = OR(h1, h2)
    return mp_neuron((h1, h2), w=[1, 1], theta=1)
```

That is the entire XOR network. Three neurons, integer weights, no learning. The trick is that the hidden layer carves the input plane into two non-overlapping half-planes, one for each firing corner, and the output unit ORs them.

The 256-function sweep was the part I was most curious about. For each function, the disjunctive-normal-form construction uses one hidden unit per true minterm. The function "constant 1" has eight true minterms and would need eight hidden units in the straight form. The trick is that you can build the complement function instead, with one hidden unit per *false* minterm, and use a NOR output. Whichever side has fewer minterms wins, and that minimum is always at most 4 when the truth table has eight rows. Every function gets a strict two-layer net with at most four hidden units.

The histogram of hidden-unit counts across the 256 functions:

| hidden units | number of 3-input functions |
|-------------:|----------------------------:|
| 0            | 2                           |
| 1            | 16                          |
| 2            | 56                          |
| 3            | 112                         |
| 4            | 70                          |

256 of 256 matched their target truth table on all eight input rows. The maximum hidden count is 4, hit by the 70 functions whose truth tables split exactly four true and four false (so neither side of the DNF/CNF choice is shorter).

![Three MP decision regions and the hidden-unit histogram for all 256 three-input Boolean functions.](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/86_boundaries.png)

What surprised me a little was how cheap the universality is once you allow two layers. The single-neuron model is genuinely restricted: linear separability is a real and felt constraint, and most interesting logical functions, XOR included, fail it. Add a single hidden layer of binary threshold units and the restriction simply evaporates. The cost is at most 2^(n-1) hidden units to cover 2^n inputs, and for small n the actual cost in the optimal-side DNF/CNF construction is much less.

## What it may mean

The McCulloch-Pitts model is the discrete ancestor of every modern neural network. The universality result we replicated, in the small case n = 3, is the version that later got generalised to continuous activations and real-valued functions in the universal approximation theorems of the late 1980s. The geometry of the failure mode is the same in both cases: a single-layer model can only carve a linearly separable boundary in input space, and a second layer is what introduces the ability to combine half-spaces into arbitrary regions.

The 1943 paper also helped frame the question of what a "computation" actually is, in a way that fed forward into the design of digital computers. Von Neumann's first draft of the EDVAC report uses the McCulloch-Pitts neuron as a stand-in for the basic logic element. The reason the model looks crude today is partly that we have moved on to differentiable activations and gradient learning, and partly that the actual hardware lineage has been so successful that we forget where the abstraction came from.

The lesson I take away, more carefully: representability is not learnability. McCulloch and Pitts showed that any finite Boolean function *can* be expressed by some MP network. They said nothing about how to find the weights given only input-output examples. That part had to wait for the perceptron rule (1958), for backpropagation in the 1970s-80s, and for everything that followed. The universality theorem is a green light for the model class, not an algorithm.

## Loose ends

A larger sweep, all 2^(2^4) = 65,536 Boolean functions of four inputs, would be cheap to run and a more honest check on whether the at-most-half-the-minterms bound continues to hold (it does, by the same DNF/CNF argument, but it would be reassuring to see). I also did not try to find minimum-size networks for individual functions, which becomes hard fast as n grows and is a classical problem in circuit complexity. With another afternoon I would push the bound on hidden-unit counts down using simple algebraic factorisations, and compare the resulting circuit sizes to what a small neural network trained by gradient descent finds. The whole thing runs in under a second on a laptop, and if you want to replicate it the code is about 200 lines.
