# I ran Wolfram's Rule 30 for 200,000 steps. The center column still looks like white noise.

*Part 29 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Stephen Wolfram wrote down Rule 30 in 1983 as one of 256 candidates for a "simplest interesting cellular automaton". The setup is almost insultingly small. You have an infinite row of cells, each cell is 0 or 1, and at every tick a cell's next value is a function of its own current value and the values of its two neighbors. There are 2^3 = 8 possible neighborhoods, each can produce a 0 or a 1, so there are 2^8 = 256 such rules. Wolfram numbered them by the 8-bit lookup table. Rule 30 is the rule whose table is binary 00011110.

He started Rule 30 from a single live cell sitting in a field of zeros, ran it for a while, and noticed something he could not explain. Most of the patterns in the resulting time-space diagram looked structured. Diagonals, triangles, repeating wedges. But the central vertical strip, the column of cells sitting directly above the original live cell, looked completely random. By 1985 he was writing in the *Journal of Statistical Physics* that this column passed every randomness test he tried. By 1988, Rule 30 was the default random-number generator inside Mathematica, and it stayed that way until 2014, when Wolfram Research finally switched to a more standard cryptographic-grade stream. Wolfram has since put up a $30,000 prize for proving certain properties of the column.

I had run small cellular automata in undergrad. I had never sat down and asked, in 2026, with a fast machine: does the central-column claim actually hold up to a real battery of tests, and does it hold up better than other "complicated" elementary CAs?

## What I tried

The plan was concrete. Pick three rules: 30 (the claim under test), 90 (the XOR rule, linear over GF(2), should be predictable), and 110 (Turing-complete in the Cook 2004 sense, complicated but not classically random). Start each from the same single-cell seed. Pad the lattice wide enough that the light cone never reaches the edge across the whole run, so the boundary cannot leak in. Record the value of the center cell at each time step. Compare the resulting binary sequence against a numpy `default_rng` draw of the same length.

The first version of the script tried 10^6 steps. That meant a lattice 2,000,200 cells wide, which is fine in memory but the per-step cost of doing eight neighborhood masks in numpy on a 2-million-cell row was slow enough that the run did not complete in the time I had set aside. I rewrote the inner loop. Bit-pack each row into `uint64` words (one word holds 64 cells), and write each rule out as a direct boolean expression on shifted copies of the row. Rule 30 turns out to be a beautiful one-liner: `new = left XOR (center OR right)`. With bit-packing and that closed form, a single time step touches 6,254 64-bit words instead of 400,256 individual cells. The whole 200,000-step simulation for Rule 30 finished in about 5 seconds. I dropped to 200,000 steps and called it good.

The randomness battery itself was deliberately old-school. Monobit (count of ones), chi-square on the byte-level distribution (256 bins), the NIST-style runs test, autocorrelation at lags 1 through 50, zlib compression ratio as a proxy for Lempel-Ziv compressibility, and Shannon entropy of k-grams for k = 1 to 8. No fancy cryptographic tests, no spectral tests. The question I cared about was: does the simple stuff catch anything?

```python
# bit-packed Rule 30 step on one row of uint64 words
def step_rule30(row):
    left = (row << 1)
    left[1:] |= (row[:-1] >> 63) & 1
    right = (row >> 1)
    right[:-1] |= (row[1:] & 1) << 63
    return left ^ (row | right)

# extract center cell over time
for t in range(1, n_steps):
    row = step_rule30(row)
    center[t] = (row[center_word] >> center_bit) & 1
```

The 200,000-bit sequence took up 25 kilobytes packed. I compared it to a Rule 90 run, a Rule 110 run, and a numpy random sequence of the same length.

## What happened

Rule 30 looked like noise. Every test came back with the same answer.

| Test | Rule 30 | numpy random |
|---|---:|---:|
| proportion of 1s | 0.5004 | 0.4992 |
| monobit p-value | 0.747 | 0.488 |
| chi-square on bytes (p) | 0.779 | 0.913 |
| runs test (p) | 0.681 | 0.960 |
| max abs autocorr (lags 1 to 50) | 0.0054 | 0.0067 |
| zlib compression ratio | 1.00064 | 1.00064 |
| Shannon H₈ / 8 | 0.9999 | 0.9999 |

The compression ratio of slightly over 1.0 is the zlib header overhead and shows up identically for the numpy stream. The autocorrelations are tiny across all 50 lags, and Rule 30 is if anything cleaner on max autocorrelation than the PRNG (0.0054 vs 0.0067), which is just sampling noise at this length. To three decimals, you cannot tell the Rule 30 center column apart from a numpy uniform binary draw using these tests.

Rule 90 and Rule 110, as a sanity check, both failed catastrophically. From the single-cell seed, Rule 90's center column is 1 at step 0 and 0 forever after. Rule 110's center column is 0 at step 0 and 1 forever after. They compress to 184 bytes out of 25,000, a ratio of 0.0018, which is the all-zeros or all-ones limit. This is not really a fair fight, since with that initial condition the center cell of Rule 90 sits at a fixed point of the XOR (its neighbors always match), and Rule 110 starts a stable rightward 1-cone that engulfs the center after one step. The lesson there is mostly that the test battery can tell a non-random sequence from a random one when one shows up. It also says that being Turing-complete or being algebraically complex (linear over GF(2)) does not on its own get you a random-looking time series at a single spatial point. Rule 30 may be doing something genuinely different.

A few caveats on these numbers. 200,000 steps is short. The NIST STS battery technically wants 10^6 bits and the diehard battery wants more. The Shannon entropy was computed only up to 8-grams, so any periodic structure with period longer than 8 (or any non-Markovian dependence at longer range) would not be caught. The compression test uses zlib, which is LZ77 with Huffman, not a strict Lempel-Ziv-Welch encoder. And we ran a single seed (the canonical single-cell start). Running with random initial conditions might in principle look different, although Wolfram's specific claim is for this particular seed.

What I did not do, and what no one has done yet, is rule out *any* deterministic structure in the column. That is the substance of Wolfram's $30,000 prize. The prize asks roughly whether the column is eventually periodic, whether the proportion of 1s tends to 1/2 in the limit, and whether there is any finite-state shortcut for predicting bit k+1 from bits 1..k. We tested none of those. We only tested whether the distribution and short-range correlations match those of a noise process at n = 200,000.

## What it may mean

The deflationary reading is: Rule 30 passes the obvious tests, which is what every PRNG passes by construction. So what.

The slightly more interesting reading is that the obvious tests are passed by an arithmetic recipe so small you can write it on a napkin: take three adjacent bits, output their XOR-with-OR. There is no entropy source, no nonlinear mixing function carefully designed by a cryptographer, no large state. The state is the universe up to the light cone at time t, but the rule for the local update is six tokens long. And the output of the rule, sampled at a single fixed spatial position, looks indistinguishable from `default_rng` to second-order statistics. That seems to suggest something about how cheap "random-looking" is, computationally. It does not need much.

It does not say Rule 30 is a *good* PRNG by 2026 cryptographic standards. Mathematica replaced it for a reason. There are tests it would presumably fail, and a sufficiently determined attacker who knew the recipe could in principle reconstruct the state. What it says, weakly, is that you can get past the first few rings of randomness testing with three bits and an XOR-OR.

## Loose ends

The thing I would do with another week is run a single longer simulation (maybe 10^7 steps), apply the full NIST STS battery, and try a few non-canonical seeds to see whether the apparent randomness is a property of the rule or of the specific single-cell start. I would also want to try the diagonal column rather than the central one, since Wolfram's later work suggests some diagonals look more structured. If anyone with a small computer cluster has run the proper STS on Rule 30 at 10^9 bits, I would love to see the failure modes. The exact form of any structure that survives would be a real result.
