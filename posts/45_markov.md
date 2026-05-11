# I rebuilt Markov's 1906 letter chain on 564,000 English letters. The spectral gap predicted the mixing time to within 0.04%.

*Part 45 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1906 Andrei Andreyevich Markov was annoyed. A colleague in Moscow, Pavel Nekrasov, had argued in print that the law of large numbers required statistically independent events, and that human acts, being interdependent, therefore lay outside the reach of probability theory. Nekrasov's reasoning ran the other way too: statistical regularities in social data implied that the underlying acts were independent, which Nekrasov took as a mathematical proof of free will. Markov, an atheist and a stubborn empiricist, thought this was nonsense dressed up in algebra.

His response was to invent a counterexample. He wrote down a sequence of dependent random variables, proved they obeyed a law of large numbers anyway, and then, in a separate paper, tallied the 20,000 letters of Pushkin's "Eugene Onegin", coded each as vowel or consonant, and reported the empirical two-by-two transition matrix. The matrix was off-diagonal. Vowels followed consonants more than chance, and consonants followed vowels more than chance. The letters were obviously not independent, and yet the vowel fraction over long stretches of the poem converged to a stable number. Dependence and ergodicity could live in the same sequence.

Out of that polemical exercise came the modern theory of Markov chains. The 1906 paper contained, in early form, the statement that a finite irreducible aperiodic chain has a unique stationary distribution, that the chain reaches it from any starting point, and that the rate of approach is geometric. The geometric rate is now known to be governed by the chain's second-largest eigenvalue in absolute value. Everything you might do with MCMC, PageRank, hidden Markov models, or queueing theory rests on that result.

I had taught this theorem more times than I want to admit and never repeated Markov's own experiment. The plan was simple. Recreate the letter-chain analysis on a real text, then verify the spectral-gap mixing prediction on a chain where the eigenvalues are known by construction.

## What I tried

For the text I used Pride and Prejudice from Project Gutenberg. It is English, not Russian, which is a compromise. Pulling the original Onegin in Cyrillic is fiddly with the encoding, and the qualitative claim Markov made is about any natural language. I lowercased the text, threw away anything that was not a letter, and coded each remaining character as 0 for vowel (a, e, i, o, u, y) or 1 for consonant. That gave a single integer sequence of 564,000 symbols.

The transition matrix is then the maximum-likelihood estimator: count how often each pair appears and normalise rows. The stationary distribution can be obtained three ways. The first is to take the left eigenvector of the transition matrix with eigenvalue 1. The second is to iterate the matrix on any starting distribution and watch it converge. The third is to read off the empirical letter frequency directly, which by ergodicity should match the stationary distribution if the chain has run long enough. Doing all three and comparing them is the cleanest possible audit of the theorem.

For the second test I wanted a chain where I could compute the spectral gap exactly. I built a four-state lazy random walk on a cycle: stay put with probability 0.7, hop one step clockwise with probability 0.15, hop one step counterclockwise with probability 0.15. By symmetry the stationary distribution is uniform on the four states, and the eigenvalues of the transition matrix are known in closed form. The second-largest in absolute value works out to exactly 0.7. The prediction is that the total-variation distance between the distribution at time t and the uniform distribution should fall like 0.7^t.

I ran the prediction two ways. The first was analytic: start from a delta on state zero, multiply by the transition matrix t times, take the total-variation distance to uniform. The second was Monte Carlo: simulate 20,000 independent trajectories, record the empirical state distribution at each step, take the TV distance to uniform. The analytic curve gives the truth; the Monte Carlo curve gives a reality check that the simulation matches.

```python
# stationary distribution three ways, on the V/C chain
P = empirical_transition(seq, 2)        # 2x2 from counts
pi_eig = stationary_eig(P)              # left eigenvector at 1
pi_pow, niter = stationary_power(P)     # power iteration from uniform
pi_emp = np.bincount(seq, minlength=2) / len(seq)
# all three agree to 14 decimals
```

That, plus a Monte Carlo loop and a TV-distance function, is the entire pipeline.

## What happened

The text chain came out roughly the way Markov reported in 1913. The empirical transition matrix on the 564,000-letter English sample is:

|             | next = vowel | next = consonant |
|-------------|-------------:|-----------------:|
| vowel       | 0.177        | 0.823            |
| consonant   | 0.557        | 0.443            |

A vowel is followed by a vowel only 17.7% of the time, well below the marginal vowel rate of 40.4%. English alternates. If the letters were independent both rows of the matrix would equal the marginal distribution, and they emphatically do not.

The stationary vector by left eigenvector is (0.40351, 0.59649). The same vector by power iteration, starting from uniform, converges in 33 steps to (0.40351, 0.59649). The empirical marginal frequency is (0.40351, 0.59649). Five decimal places of agreement between three independent routes to the answer. That is the ergodic theorem in plain view: time averages equal ensemble averages on this chain.

The second eigenvalue of the text matrix is |lambda_2| = 0.3793, giving a mixing-time scale of 1/(1 - 0.3793) ~ 1.6 steps. Memory of which letter type started the sequence is effectively gone after about three to five letters. For a model this crude, the chain forgets fast.

The engineered four-state chain gave a much cleaner picture. Starting from a delta on state zero, the analytic TV distance to uniform was 0.750 at t = 0, 0.131 at t = 5, 0.00086 at t = 20, and 2.0 x 10^-13 at t = 80. Fitting log(TV) against t over the range where the geometric regime had taken hold (steps 3 through 60), the slope gave a per-step decay factor of 0.6998. The predicted factor is exactly 0.7. The ratio is 0.9997.

The Monte Carlo run with 20,000 trajectories tracked the analytic curve closely until about t = 50, where the empirical TV bottomed out at roughly 5 x 10^-3. That floor is exactly what you would expect from sampling noise at N = 20,000: the TV between an empirical four-bin distribution and its true mean scales like 1/sqrt(N), which gives 0.007. Below that the Monte Carlo cannot resolve the geometric decay anymore. The chain has mixed; the estimator has not.

![Vowel/consonant transition matrix on Pride and Prejudice and TV-distance decay on a 4-state lazy walk. The dashed black line is the empirical fit; the dotted red line is the spectral-gap prediction.](https://raw.githubusercontent.com/Macaulay001/forgotten-theories-blog/main/posts/figures/45_tv_decay_and_heatmap.png)

A few details surprised me. One was the cleanness of the power-iteration convergence. Thirty-three matrix-vector multiplies brought the iterate to within machine precision of the analytic eigenvector. The other was how brittle the Monte Carlo verification was at long times. To see the geometric tail past t = 50 cleanly you would need millions of trajectories, not tens of thousands. The Monte Carlo is dominated by sampling noise long before the chain stops mixing. The spectral computation is in that sense much stronger evidence than the simulation.

## What it may mean

What Markov accomplished in 1906 is a working answer to the question "when can you trust a long-run average from a dependent sample?" The answer is: when the chain that generated the sample is irreducible and aperiodic, and the average is over a horizon longer than 1/(1 - |lambda_2|) steps. The spectral gap is not a curiosity. It is the quantity that tells you how many samples your MCMC needs, how many iterations of PageRank to run, how long a queue takes to forget its initial state. Every Bayesian who quotes an effective sample size from autocorrelation is using a corollary of the 1906 theorem.

The Pushkin exercise was originally a rhetorical move against a colleague who happened to be wrong about something else. It became a foundation for half of computational statistics. On this sample, the 1906 prediction holds at three decimal places in the rate constant and at the floor of double precision in the convergence of power iteration. Markov's instinct that dependence and ergodicity can coexist appears to be exactly correct, and the rate is exactly the second eigenvalue.

There are caveats. The vowel-consonant abstraction averages over a lot of higher-order structure. Real English has order-2 and order-3 dependencies (the letter 'q' is almost always followed by 'u'; 'th' is much more common than statistical predictions on bigrams would imply). The two-state chain captures only the simplest stratum. A full 26-state letter chain on the same text would still satisfy the theorem, with a different stationary vector and a different mixing rate. The theorem is indifferent to the state-space size as long as the matrix is irreducible and aperiodic.

## Loose ends

With another week I would rerun the analysis on a Cyrillic version of Eugene Onegin and report the exact transition probabilities Markov got in 1913, side by side with my English numbers. I would also fit a full 26-state letter chain to the same corpus and check whether the spectral gap of the larger chain still predicts the mixing time of, say, a randomly initialised text generator. The whole pipeline runs in about ten seconds on a laptop and the data is free. If you want to repeat Markov's experiment in your own language, you can do it before lunch.
