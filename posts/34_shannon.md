# I tested Shannon's 1948 channel limit on a noisy bit pipe. The wall is exactly where he said.

*Part 34 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Claude Shannon published *A Mathematical Theory of Communication* in
1948 and, in one paper, founded an entire field. The headline result
was strange to engineers at the time. He claimed that every noisy
channel has a number attached to it, the capacity C, and that you can
transmit bits at any rate below C with as little error as you like, if
you are willing to use long enough codes. Above C, you cannot, no
matter how clever the code. People found the converse half hard to
accept. There was a feeling that surely, with enough engineering,
noise could be beaten. Shannon said no. The line is sharp and he gave
you the formula.

For the binary symmetric channel, where every transmitted bit flips
independently with probability p, the capacity is

  C(p) = 1 + p log2(p) + (1 - p) log2(1 - p)

I wanted to see the wall for myself, not as a theorem but as a curve
fit by simulated data. Old result, yes. Still worth poking.

## What I tried

The plan is simple enough to fit on an index card. Pick a flip
probability p. Generate a long stream of random information bits.
Encode them with some block code. Push the codeword through a
simulated BSC. Decode. Count how many of the original information
bits came out wrong. Repeat across rates and across p, and plot the
empirical points against Shannon's curve.

I used two code families because they bracket the interesting cases.

The first is the repetition code of odd length n. To send one bit, you
send it n times, and the receiver takes a majority vote. The rate is
1/n, so larger n means more reliability and less throughput. This is
the most pedestrian code in the world. It is also the easiest to
imagine: triple-redundancy, five-fold, eleven-fold, and so on.

The second is Hamming(7,4), an actual single-error-correcting linear
code that fits four information bits into a seven-bit codeword. Its
rate is 4/7, roughly 0.571. It is the simplest non-trivial example of
what coding theory actually looks like. It has a generator matrix and
a parity-check matrix and a syndrome decoder. I implemented it from
scratch because the whole point is to see the machinery.

For each combination of code and p, I sent at least 200,000 information
bits through the channel and computed the post-decode bit error rate.
The full simulation runs in under a minute on a laptop.

```python
G = np.array([[1,0,0,0, 1,1,0],
              [0,1,0,0, 1,0,1],
              [0,0,1,0, 0,1,1],
              [0,0,0,1, 1,1,1]], dtype=np.uint8)
msgs = rng.integers(0, 2, size=(N, 4)).astype(np.uint8)
codewords = (msgs @ G) % 2
flips = (rng.random(codewords.shape) < p).astype(np.uint8)
rx = codewords ^ flips
synd = (rx @ Hmat.T) % 2
for i, s in enumerate(map(tuple, synd)):
    pos = SYND.get(s, -1)
    if pos >= 0: rx[i, pos] ^= 1
```

That is the heart of it. Encode by matrix multiply mod 2, flip bits at
random, decode by looking up the syndrome.

## What happened

The wall showed up sharp and on time.

Start with p = 0.01, a very clean channel. Capacity is about 0.919
bits per channel use. Every code I tried worked fine. The uncoded BER
is 1%, the repetition-3 BER drops to 3.3e-4, repetition-11 to under
1e-5. Hamming(7,4) gets 7.8e-4. Nothing surprising.

Walk p up to 0.10. Capacity now is 0.531. Hamming(7,4), with its
fixed rate of 0.571, is sitting just barely above the wall. Its
post-decode BER is 6.7%, only modestly better than the raw 10%. The
correction is real but limited. Meanwhile repetition-11, at rate
0.091, sits comfortably below the wall and gets 3.6e-4.

Now push to p = 0.20. Capacity collapses to 0.278. Hamming(7,4) is
now way above its rate ceiling. Its BER is 0.197, almost identical
to the channel itself. The syndrome decoder is confidently
mis-correcting about as often as it correctly corrects.

At p = 0.25, capacity is 0.189. Hamming(7,4) returns BER = 0.262,
which is worse than the uncoded channel. The decoder is now actively
hurting you. This is the converse half of Shannon's theorem in
miniature. If your rate exceeds C, no decoding scheme can win,
because there is not enough mutual information in the received
sequence to identify the codeword.

| p    | C(p)  | Hamming(7,4) BER | Uncoded BER |
|------|-------|------------------|-------------|
| 0.05 | 0.714 | 0.020            | 0.050       |
| 0.10 | 0.531 | 0.067            | 0.100       |
| 0.15 | 0.390 | 0.128            | 0.150       |
| 0.20 | 0.278 | 0.197            | 0.200       |
| 0.25 | 0.189 | 0.262            | 0.250       |

The crossover for Hamming(7,4) happens around p = 0.087, where C(p)
equals the code rate of 4/7. The simulation lands on that prediction
within the noise of 100,000 blocks.

Meanwhile the repetition codes draw a clean trajectory parallel to
the capacity curve. At every p, the n = 11 code (rate 0.091) wins
the lowest BER, the n = 9 (rate 0.111) is next, and so on. None of
them have rates anywhere near C(p) for small p, so they are
suboptimal in a different way. They are wasteful with bandwidth but
they never blow up.

A practitioner would call this the throughput-reliability tradeoff,
and the Shannon curve is the Pareto front of what is achievable. In
the figure I generated, the achievable region (R < C(p)) is shaded
green and the forbidden region (R > C(p)) is shaded red. Every
sub-1%-BER point lands in green; the Hamming failure points (BER >
5%) land in red. The boundary is doing real work.

## What it may mean

Nothing in this exercise is news. Shannon's theorem is one of the most
verified results in applied mathematics, and modern codes like LDPC
and polar get within fractions of a decibel of the limit. The
Voyager 2 link from Neptune in 1989 ran at about 1.7 dB above
Shannon, using concatenated Reed-Solomon and convolutional coding.
Modern Wi-Fi is similarly close.

What may be worth pausing on is how cleanly the wall shows up even in
a toy simulation. There is no fitted parameter. The capacity curve
falls out of the entropy formula and a half-page proof, and the
empirical points respect it without exception. Hamming(7,4)
specifically tells you something interesting about cheap codes: they
are useful exactly until the channel gets noisy enough that their
rate exceeds capacity, and not one bit further. A short-block
single-error-correcting code does not survive truly noisy channels,
it fails in a sharp and predictable way.

If you were designing a real link today, the lesson would be that the
fundamental limit is non-negotiable, but the gap between practical
codes and the limit is now small. Almost all the engineering effort
sits in the last 1 to 2 dB.

## Loose ends

The simulation does not include AWGN, where C = B log2(1 + S/N) and
soft-decision decoding matters. It does not include LDPC or turbo
codes, the families that actually approach Shannon. The finite
block-length corrections (Polyanskiy-Poor-Verdu, 2010) say that the
gap to capacity at moderate block length is not negligible, and a
follow-up could plot empirical points against the finite-n bound
rather than the asymptotic capacity. With another week I would code
up a rate-1/2 LDPC under belief propagation and watch it climb
within half a decibel of the wall. The full notebook reproduces in
under a minute.
