# Ancel Keys reshaped 50 years of dietary advice. Modern meta-analyses may not agree.

*Part 18 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1955, Ancel Keys stood at a WHO meeting in Geneva and laid out what he thought was killing American men: saturated fat. His Seven Countries Study, launched soon after, pulled cohort data from Finland, Italy, Greece, the Netherlands, Japan, the United States, and Yugoslavia, and reported a clean correlation between saturated fat intake, blood cholesterol, and coronary heart disease mortality. The chain looked simple. Eat butter and cream, raise serum cholesterol, get a heart attack.

By the late 1970s that chain had become US dietary policy. The 1977 McGovern report and the 1980 Dietary Guidelines told Americans to cut saturated fat. Food manufacturers swapped fat for sugar and refined starch. The hypothesis was contested from the start (John Yudkin pointed at sugar, Pete Ahrens at carbohydrate, others at study selection) but Keys's framing won the policy fight. I wanted to see what the published literature looks like now, several decades later, and whether the evidence base has stayed where Keys left it.

## What I tried

The plan was small and unglamorous. I ran eight PubMed queries aimed at the saturated-fat / cardiovascular-disease question, pulled the abstracts that came back, and used an LLM to classify each one for study type and reported direction. Nothing in this design tests the underlying biology. It tests what the literature, as written, says.

The queries were chosen to span the way different communities phrase the question:

- "saturated fat" cardiovascular
- "saturated fatty acid" mortality
- "dietary fat" coronary heart disease
- "diet-heart" hypothesis
- PURE study dietary fat
- low-fat diet cardiovascular events
- Mediterranean diet randomized
- saturated fat LDL meta-analysis

After deduping PMIDs across queries I had 561 abstracts. I sent each to GPT-4o-mini with a strict JSON prompt asking for four things: whether the paper is actually on-topic (studies dietary SFA and CVD outcomes), the study design (RCT, meta-analysis, prospective cohort, case-control, ecological, mechanistic review, other), the direction of the reported finding (supports SFA → ↑CVD, null, inverse, context-dependent, or agnostic), and the rough endpoint. I cached results and ran 16 abstracts in parallel.

```python
PROMPT = """Classify a biomedical abstract on SFA and CVD.
Output STRICT JSON: on_topic, study_type, direction, endpoint, year_int.
direction in {supports_lipid_heart, null_or_no_association,
              inverse, context_dependent, agnostic}
Title: {title}
Abstract: {abstract}
JSON only:"""

for it in todo:
    r = call([{"role":"user","content": PROMPT.format(**it)}],
             model="openai/gpt-4o-mini", max_tokens=250)
    d = json.loads(r["text"])
    out.write(json.dumps(d) + "\n")
```

A few things to be honest about up front. I did not read every abstract by hand. The classifier may misread a paper whose conclusion section walks back a strong title. "On-topic" is also a judgment call: I let the LLM apply it. And PubMed indexing is biased toward English-language and recent work, so the 1980s row in my table is genuinely thin. Take the early-decade numbers as suggestive, not definitive.

## What happened

Of the 561 abstracts, the classifier flagged 158 as actually on-topic for the SFA-CVD question. The overall distribution surprised me a little. I had expected either a clean majority of "supports" papers (which would have meant the literature still backs Keys) or a clean majority of "null" papers (which would have meant the field had simply moved on). Instead the largest single bucket was "supports," at 33.5%, with "context-dependent" close behind at 31.6% and "null" at 24.7%. About 6% reported inverse associations. Roughly 4% were agnostic.

That overall split hides the more interesting structure. When I broke the 158 papers by study design, the picture shifted:

| Study type           | n  | Supports | Null | Context | Inverse |
|----------------------|---:|---------:|-----:|--------:|--------:|
| Prospective cohort   | 37 | 57%      | 8%   | 14%     | 19%     |
| Meta-analysis        | 36 | 17%      | 39%  | 39%     | 3%      |
| RCT                  | 21 | 48%      | 29%  | 24%     | 0%      |
| Mechanistic review   | 27 | 26%      | 33%  | 37%     | 0%      |
| Case-control         | 3  | 67%      | 33%  | 0%      | 0%      |

The meta-analyses, which sit at the top of most evidence hierarchies, are the *least* supportive of Keys's hypothesis in this sample. Only 17% of them support a simple SFA → CVD relationship. The plurality, split evenly between "null" and "context-dependent," suggest the effect either is not there in pooled data or depends on what the saturated fat is replaced with. Prospective cohorts, by contrast, look much more Keys-friendly at 57% supporting. RCTs sit in the middle.

I do not want to overread that gap. Cohorts and meta-analyses ask slightly different questions, and meta-analyses tend to pool heterogeneous designs and report the noisiest possible average. The point is just that as you climb the evidence pyramid in this dataset, support for the classical lipid-heart hypothesis appears to weaken.

By decade, support drifts downward:

| Decade | n  | Supports | Null | Context | Inverse |
|--------|---:|---------:|-----:|--------:|--------:|
| 1980s  | 5  | 80%      | 20%  | 0%      | 0%      |
| 1990s  | 6  | 33%      | 33%  | 33%     | 0%      |
| 2000s  | 7  | 57%      | 14%  | 29%     | 0%      |
| 2010s  | 53 | 34%      | 26%  | 34%     | 4%      |
| 2020s  | 87 | 29%      | 24%  | 32%     | 8%      |

The 1980s row is five papers. I would not hang a thesis on it. But the 2010s and 2020s rows have 53 and 87 papers, and "supports" sits at roughly a third. "Context-dependent" papers, almost absent in the 1980s sample, are about a third of the modern literature. The inverse category, very rare historically, shows up in 8% of 2020s papers.

The pattern across both cuts points the same way. The most recent and the most synthetic evidence (meta-analyses and 2020s output) is the least likely to endorse the simple version of Keys. The remaining support is concentrated in prospective cohorts, which is roughly the design Keys himself ran.

## What it may mean

A few hedges before anything else. This is not a meta-analysis. It is a literature survey of abstract-level conclusions, classified by an LLM I did not audit paper-by-paper. The categories are coarse. "Supports" lumps together a strong association with a weak one. "Context-dependent" lumps together "SFA is fine if you replace it with unsaturated fat" and "SFA is fine if you replace it with protein but not refined carbs" and several other distinct claims. There is also publication-selection bias I cannot correct for from outside.

With those caveats in place, what the data appears to suggest is this. The strong form of the lipid-heart hypothesis (more SFA → more CHD, full stop) does not look like the modal finding in the modern literature on this sample. The modal finding looks closer to "it depends what you eat instead." That is consistent with what the 2015 US Dietary Guidelines Advisory Committee report acknowledged, when blanket SFA caps were softened in favor of focusing on overall dietary pattern. It is also roughly consistent with the PURE study findings published the same decade. None of that proves Keys was wrong; the cohort evidence still leans his direction. It does suggest the policy-relevant question may be subtler than the version that got written into food labels in 1980.

I want to be careful here. I am not making any recommendation about what anyone should eat. This is a survey of how a literature has voted, not advice. I do not know the right diet. The thing I think I can say, with appropriate hedging, is that the modern evidence base looks more mixed than the public memory of "saturated fat is bad for your heart" implies.

## Loose ends

The obvious next step is to actually run a meta-analysis on the meta-analyses, pulling effect sizes rather than abstract-level direction calls, and adjusting for replacement nutrient. A second step would be to separate the LDL-particle subtype literature, since pooled LDL-cholesterol may hide important structure. A third would be to hand-audit a random sample of the 158 classifications to see how often GPT-4o-mini gets the direction wrong on this exact task. If anyone has a curated SFA-CVD effect-size table with replacement-nutrient metadata, I would like to see it. That dataset could be assembled by one person in a couple of weeks.
