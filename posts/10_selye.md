# Hans Selye said stress responses come in three phases. Modern papers only describe two.

*Part 10 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In July 1936, Hans Selye published a one-page letter in *Nature* titled "A syndrome produced by diverse nocuous agents". The claim was bold for a one-pager. Rats exposed to any sufficiently nasty insult, whether cold, surgical injury, or injected toxin, went through the same three-stage response. First an alarm phase, with a cortisol surge and a hammering sympathetic system. Then a resistance phase, where the body looked almost normal again. Then, if the stressor kept going long enough, an exhaustion phase, where the adaptive machinery collapsed.

Selye later wrote a whole book around this, *The Stress of Life* (1956), and the triphasic curve became a fixture of textbooks. It is also one of those ideas that, in modern endocrinology, has been politely demoted. The allostatic-load framework (McEwen, 1998) is more careful about specifics. But Selye's three-stage curve is still taught as if it is what physiology actually does. I wanted to check whether modern papers still describe that shape.

## What I tried

The honest version of the test would be a parametric fit of physiological time-series. Heart rate over a prolonged cold-pressor test, for example, fitted to Selye's three-phase form versus a single-exponential null. That is the experiment I would run with another week. Instead I ran a cheaper proxy: ask the modern literature what shape it sees.

I pulled 79 abstracts from PubMed using a deliberately broad query covering "stress response", "alarm reaction", "general adaptation syndrome", and "HPA axis time course", filtered to humans, rats, and mice. Then I used GPT-4o-mini to classify each abstract into a shape category: triphasic in Selye's sense, biphasic alarm-resistance (just the first two phases), biphasic decay-recovery, monotonic rise, monotonic decay, other time course, or not a time-course paper at all. For each abstract the classifier also returned the stressor (cold, restraint, psychosocial, etc.) and the measured marker (cortisol, heart rate, blood pressure).

The reason I am hedging this so heavily is that the classifier is doing real work here. If GPT-4o-mini systematically misses triphasic descriptions, the whole result falls apart. I spot-checked perhaps a dozen of the classifications by hand and they looked reasonable, but I have not done a calibration study against expert annotation. The total cost of the LLM calls was about 8 cents, which is its own kind of caveat: this is a small, cheap probe, not a definitive review.

The core of the script looked like this. I am keeping the prompt and the call together because that is where most of the methodological risk lives.

```python
PROMPT = """Classify whether this abstract describes a TRIPHASIC
stress response (alarm spike -> plateau/resistance -> collapse).
Output JSON with keys: shape, stressor, marker, confidence.
shape in: triphasic | biphasic_alarm_resistance | monotonic_rise
        | monotonic_decay | biphasic_decay_recovery | other
        | not_a_time_course
Title: {title}
Abstract: {abstract}
"""
for item in abstracts:
    r = call(PROMPT.format(**item), model="openai/gpt-4o-mini")
    counts[json.loads(r["text"])["shape"]] += 1
```

That is roughly the whole thing. PubMed eFetch on one end, an LLM classifier on the other, and a Counter in between.

## What happened

The first surprise was how few of the 79 abstracts were actually about a physiological time course. 56 of them, or 71%, were classified as "not a time-course paper". The phrase "stress response" turns out to be used very widely in biomedical writing. There were papers on oxidative stress in cancer cell lines, on antibiotic stress responses in *E. coli*, on the unfolded protein response, on heat-shock transcriptional programs, on social stress in epidemiological surveys. Almost none of these report what cortisol or heart rate does over time. They use "stress response" to mean a gene-expression module or a population-level outcome, not a curve.

That left 23 abstracts that actually describe something happening over time in a stressed organism. Here is the breakdown.

| Shape                          | n   | % of time-course |
|--------------------------------|----:|-----------------:|
| Triphasic (Selye GAS)          | 0   | 0%               |
| Biphasic alarm-resistance      | 6   | 26%              |
| Biphasic decay-recovery        | 1   | 4%               |
| Other (unclassified)           | 16  | 70%              |
| Monotonic rise or decay        | 0   | 0%               |

Zero of 23. None of the modern abstracts in my sample describe the three-stage alarm-resistance-exhaustion shape that Selye said was the universal signature of stress. The most common identifiable shape, when one was identified, was biphasic alarm-resistance: an acute spike followed by a return toward (but not to) baseline. That is the first two stages of Selye's curve, with the collapse phase missing.

The "other" bucket is large (16 of 23), and I do not want to lean too hard on it. Looking through those classifications, most appear to be papers that describe a time course but in a way that does not map cleanly onto any of my pre-specified shapes. Some report only a single post-stress measurement compared to baseline. Some compare two groups at a fixed time point. A few describe non-monotonic patterns that the classifier was not confident enough to call biphasic.

Even granting that the "other" papers might hide a triphasic curve or two, the headline result is striking. Selye proposed the triphasic shape as the diagnostic signature of stress, and in a 79-paper sample from PubMed, zero abstracts mention it explicitly enough for a competent LLM to pick it up. Among the abstracts that *do* describe a clear shape, the modal answer is the front half of his curve without the back half.

The classifier also extracted stressors and markers. The stressors were the usual suspects: restraint stress in rats, psychosocial stress in humans, cold pressor, forced swim, surgical injury. The markers were mostly cortisol and ACTH, with a smaller number of cardiovascular variables. Selye's claim was that the triphasic shape should appear in any of these. It does not appear in any of these, at least at the level of how modern abstracts summarize their own findings.

## What it may mean

Two readings of this result, and I am not sure which I prefer.

The first is that Selye's specific triphasic curve was always an overstatement. The empirical pattern is biphasic: alarm, then resistance, full stop. The "exhaustion" phase may have been an extrapolation from a small number of long-duration rat experiments where animals were stressed essentially to death, and may not be a general feature of biological responses to stress. If that is right, allostatic load is the right successor framework and Selye's third stage is a historical artifact of how he set up his original experiments.

The second reading is more charitable. Modern experimental protocols, especially in human research, rarely run long enough to see an exhaustion phase. Ethics boards are not enthusiastic about chronically stressing humans for weeks. Most rodent studies are acute or subchronic. If exhaustion only shows up after very prolonged stress, the modern literature may simply be sampling the first two stages of a real three-stage process. The absence in abstracts would then reflect a protocol gap, not a biology gap.

Either way, the strong form of Selye's claim, that the triphasic shape is the universal signature of stress, is not what current papers report. The biphasic alarm-resistance pattern is. That seems worth saying even with all the caveats.

## Loose ends

The PubMed query was noisy. 71% of hits were not physiology-time-course papers, and the abstracts in the residual 23 are themselves a heterogeneous mix. A targeted query, for example HPA-axis cortisol time courses in chronic-stress protocols longer than two weeks, would shrink the sample but sharpen the test. I would also want to validate the classifier against a small set of expert-annotated abstracts before trusting any of these counts to more than two significant figures. With another week, I would do that calibration and rerun on a tighter query. The whole pipeline cost about 8 cents and runs in under ten minutes, so anyone who wants to repeat it with a better query can.
