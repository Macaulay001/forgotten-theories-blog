# George Pólya in 1945: "Generalize first." On 15 math problems, GPT-4o-mini doubled its score by doing exactly that.

*Part 19 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In *How to Solve It* (Princeton, 1945), George Pólya named a small paradox that has bothered teachers ever since. He called it the Inventor's Paradox: "the more ambitious plan may have more chances of success." The harder problem, sometimes, is the easier one. If you want the sum of the first 100 integers, you might struggle. If you want the closed form for the sum of the first n integers, you derive n(n+1)/2 in two lines and then plug in 100. The general case can expose a structure that the specific case hides.

Pólya kept giving examples in his later books: stronger induction hypotheses that go through where weaker ones stall, generating-function arguments that treat a parameter as a variable, bounds that get easier when you stop pinning the parameter down. The principle is taught in classical math pedagogy, but I have never seen it tested empirically across a corpus. It always felt like one of those aphorisms that survives because it sounds nice. So I wanted to know: does it actually help, on a panel of problems, with a measurable success rate?

## What I tried

I picked 15 problems that all have a natural generalization. Closed-form sums (1·2 + 2·3 + ... + 99·100), a telescoping partial-fraction sum, the binomial sum identity, an alternating binomial sum, a geometric series, GCD(2^100 − 1, 2^60 − 1), the constant term of (1+x)^20(1+1/x)^20, a 1×10 tiling count, the last digit of 7^137, the cubes-equal-triangular-squared identity, a trig limit, a power integral, F(10), and a Vieta sum-of-roots problem. Each one has a textbook generalization sitting one step above it.

For every problem, I asked GPT-4o-mini twice, at temperature 0. Mode A: "solve this specific problem directly." Mode B: "first solve the general version, then specialize." Then a second LLM call graded each answer against the known true answer, returning yes or no. I'm using LLM-as-judge here, which is imperfect, but for arithmetic answers it tends to be reliable, and I spot-checked the calls by hand. The grader is the same model, so any systematic bias hits both modes equally.

The core of the experiment is small enough to show:

```python
for p in PROBLEMS:
    sol_A = llm_solve(PROMPT_SPECIFIC.format(problem=p['specific']))
    ok_A  = judge(p["specific_answer"], sol_A.get("answer", ""))

    sol_B = llm_solve(PROMPT_GENERAL.format(problem=p['general']))
    ok_B  = judge(p["specific_answer"], sol_B.get("answer", ""))

    out.write(json.dumps({"id": p["id"], "type": p["type"],
                          "A_correct": ok_A, "B_correct": ok_B}) + "\n")
```

That is it. 15 problems, 2 modes each, 30 solver calls plus 30 judge calls. The whole run cost cents. The interesting work was picking problems where a real generalization exists rather than a verbal restatement.

## What happened

Direct solving got 6 of 15 right. Generalize-first got 12 of 15 right.

| Outcome | Count |
|---|---|
| Both correct | 5 |
| Only generalize-first correct | 7 |
| Only direct correct | 1 |
| Neither correct | 2 |

So the success rate roughly doubles, from 40% to 80%, on this panel. Seven problems flipped from wrong-when-direct to right-when-generalized. One problem flipped the other way. The asymmetry is sharp enough that I went back and re-read the model's actual responses for the seven flipped cases, suspicious that I had stacked the deck.

I had not, as far as I can tell. In the telescoping sum 1/(1·2) + 1/(2·3) + ... + 1/(99·100), direct mode tried to add things term by term and came back with a wrong fraction. Generalize-first wrote 1/(k(k+1)) = 1/k − 1/(k+1), telescoped to 1 − 1/(n+1), and got 99/100 immediately. For 1·2 + 2·3 + ... + 99·100, direct mode produced an arithmetic slip. Generalize-first wrote k(k+1) = k^2 + k, summed using the standard formulas, got n(n+1)(n+2)/3, plugged in 99 and returned 333,300. The last digit of 7^137 was the same story: direct attempts tried partial computation, generalize-first noticed the period-4 cycle (7, 9, 3, 1), reduced 137 mod 4, and got 7.

The pattern across the seven flipped problems is consistent. The general form forces the model to write down an identity, a closed form, or a periodic structure. Once the identity is on the page, the specialization is one substitution. Direct mode, by contrast, encourages the model to compute, and it tends to slip. The constant term of (1+x)^20 (1+1/x)^20 is a clean case: generalize-first wrote it as C(2n, n) and returned C(40,20) = 137,846,528,820. Direct mode tried to expand and miscounted. The cubes identity sum k^3 = (sum k)^2 came out clean by induction at the general level, and direct computation arrived at the right number 3025 but somehow tagged it as not equal to 55^2.

The one case where generalize-first hurt was Vieta's formulas. The polynomial x^4 − 5x^3 + 6x^2 + 4x − 8 has sum of roots equal to 5, which is just the negative of the x^3 coefficient. Direct mode wrote that down. Generalize-first decided to discuss all the Vieta relations at once and ended up confusing itself about which sign and which coefficient applied. The general statement added more variables than the specific question needed.

Roughly grouped by problem type, the heuristic helped most on telescoping sums, partial-fraction sums, modular periodicity, GCD identities, binomial constant terms, and the cubes-sum identity. It was neutral on the trivial closed-form problems (1+2+...+100, 1+1/2+...+1/1024) where both modes succeed, and on the integral and the trig limit where both also succeed. It hurt only on Vieta. That is one out of fifteen.

## What it may mean

I want to be careful here. This is fifteen problems, one model, one prompt template per mode. It is suggestive, not conclusive. But the size of the effect is hard to dismiss. Doubling a correctness rate by changing the framing of the question, with no extra information given to the model, suggests Pólya was pointing at something real about how reasoning composes.

If the pattern holds at larger scale, a few things follow, hedged. A simple generalize-first wrapper around an LLM math call could meaningfully improve performance on the kind of problems that have algebraic structure underneath. Systems like AlphaProof and DeepMind's IMO-level work almost certainly already do something equivalent, often implicitly, by working at the lemma level. The heuristic might also transfer to formal proof assistants, where stating the stronger lemma is sometimes the only way to get induction to close. In curriculum design, the analogue would be teaching students to solve the harder problem on the way to the easier one, which is exactly Pólya's original pedagogical recommendation.

What I cannot rule out is that the prompt template for mode B simply elicits more careful step-by-step reasoning, and that the "generalization" framing is a proxy for chain-of-thought. If that is what is happening, the lift is real but the explanation is different. Pulling those apart would need a third mode: chain-of-thought on the specific problem with no generalization. I have not run that.

## Loose ends

Fifteen problems is small. A panel of 100, with difficulty calibrated across high school, undergraduate, and competition levels, would be more convincing. I would also want at least three models (a frontier model, a mid-tier model, a small open one) to see whether the heuristic helps the strong solvers as much as the weak ones, or whether it mostly rescues the weak. The Vieta failure is interesting on its own, and I suspect there is a small family of problems where generalization specifically hurts, probably those where the specific case has fewer free parameters than the general one. Anyone with a clean math problem set with known answers, I would like a copy.

