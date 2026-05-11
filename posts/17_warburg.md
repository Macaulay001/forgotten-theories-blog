# Warburg said cancer is a metabolic disease in 1924. A 2024 FDA approval may finally agree.

*Part 17 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

Otto Warburg published a short paper in *Klinische Wochenschrift* in 1924 with a claim that, even now, makes oncologists uncomfortable. He had measured the lactic acid output of tumor slices and noticed something off. Tumors fermented glucose to lactate even when oxygen was sitting right there, available, free. Normal cells do this only when starved of air. Tumors did it anyway.

From this Warburg made the leap that got him remembered, and then forgotten, and then half-remembered again. Damaged mitochondria, he argued, were not a side effect of cancer. They were the cause. Cripple the respiratory machinery of a cell, force it into glycolysis, and you get malignancy. He pushed this view for the rest of his life, including in a 1956 *Science* essay called "On the origin of cancer cells".

It did not survive contact with the oncogene revolution. Boveri, Bishop, Varmus, and the long parade of mutated genes that followed seemed to settle the question. Cancer was a disease of DNA. Warburg's metabolic story slid into the history-of-science footnotes.

## What I tried

I wanted to see what the modern literature actually says, not the cartoon version of it. Specifically: of the people writing about the Warburg effect today, how many still hold Warburg's *strong* causal claim, how many think metabolism is downstream of oncogenes, and how many sit in the awkward middle.

The plan was small. Eight PubMed queries, each tuned to a different facet of the question:

- `"Warburg effect"[Title/Abstract]`
- `"aerobic glycolysis" cancer`
- `"tumor metabolism" mitochondria`
- `reverse Warburg`
- `oncogene glucose uptake cancer cells`
- `IDH mutation glioma metabolism`
- `HK2 hexokinase tumor`
- `PKM2 cancer pyruvate kinase`

I capped each at 80 records, deduplicated by PMID, and pulled the abstracts via E-utilities. That left 557 abstracts. Then the hard part. Reading 557 cancer-metabolism papers by hand is not a weekend project, so I used GPT-4o-mini to classify each one. The prompt asked the model to mark whether the paper was on-topic (some HK2 papers, for instance, are really about non-cancer biology), and then to place it on a stance ladder: Warburg's strong causal claim, metabolism as a consequence of oncogenic activation, metabolism as an oncogenic *driver* (the IDH-mutant kind of story), therapeutic target with no causal commitment, or review and agnostic.

The core of it was just this:

```python
PROMPT = """Classify a cancer-metabolism abstract. JSON only.
stance: one of [
  warburg_strong_causal,    # metabolic switch CAUSES cancer
  metabolism_consequence,   # downstream of oncogenes
  metabolism_driver,        # metabolic enzymes drive (e.g. IDH)
  therapeutic_target,
  review_agnostic, other
]
key_pathway: e.g. HIF-1a, IDH1/2, PKM2, HK2"""

for item in todo:
    r = call(PROMPT.format(**item), model="openai/gpt-4o-mini")
    out.write(json.dumps(parse(r)) + "\n")
```

I want to be honest about what this means. The classifier is doing the reading. If GPT-4o-mini systematically misreads a class of papers, the numbers below will be wrong in the same way. I spot-checked 30 of them by hand and was happy enough to keep going, but a reader should hold these percentages at arm's length.

## What happened

Out of 557 abstracts, 408 came back on-topic. The rest were noise: HK2 in immune cells, PKM2 in non-cancer development, the occasional review of glycolysis that never mentioned tumors. That on-topic rate (around 73%) is roughly what I expected from this style of query.

Inside the on-topic set, the stance distribution looks like this:

| stance                                  |   n |     % |
|-----------------------------------------|----:|------:|
| therapeutic target (cause-agnostic)     | 152 | 37.3% |
| metabolism as oncogenic driver          | 112 | 27.5% |
| metabolism as consequence of oncogenes  |  98 | 24.0% |
| Warburg's strong causal claim           |  34 |  8.3% |
| review / agnostic                       |  12 |  2.9% |

A few things jump out. The strong Warburg position is alive, in roughly 8% of the modern literature, but it is a clear minority. The consequence camp (metabolism as a downstream effect of Myc, PI3K-Akt-mTOR, HIF-1α activation) holds about a quarter of the field. The biggest single chunk, 37%, treats tumor metabolism as a drug target and just declines to argue about cause. That last group may be the most honest: they want to kill cancer cells with metabolic interventions and find the etiology debate not very useful for that purpose.

The middle category is the one I find most interesting. About 27.5% of papers treat metabolism as an *oncogenic driver*, which is not the same thing as Warburg's strong claim. The strong claim was about generic mitochondrial damage. The driver position is more specific: a particular metabolic enzyme, when mutated, produces an oncometabolite that itself drives transformation. IDH1 and IDH2 are the textbook example, and the byproduct (2-hydroxyglutarate) inhibits α-ketoglutarate-dependent dioxygenases that, among other things, do DNA demethylation. The metabolism, in those tumors, really is the driver.

The pathway frequencies fit the same story:

| pathway   | mentions |
|-----------|---------:|
| HK2       |       54 |
| PKM2      |       53 |
| HIF-1α    |       22 |
| IDH1/2    |       12 |
| (Myc, PI3K-Akt-mTOR, c-Myc, AMPK trail behind)

HK2 and PKM2 are downstream glycolytic enzymes that get overexpressed in many tumors. They show up in both consequence and therapeutic-target papers. HIF-1α sits at the interface: stabilized by hypoxia, it pushes the glycolytic program. IDH1/2 are the cleanest causal story in the whole field, and they only show up 12 times because IDH-mutant cancers are relatively rare. (Grade 2 glioma, AML, cholangiocarcinoma.)

I tried to split by decade as well. Through the 1990s and most of the 2000s, "metabolism as consequence" dominated. After about 2008, which is when Vander Heiden, Cantley, and Thompson's *Science* paper reframed aerobic glycolysis as a hallmark, the driver and therapeutic-target categories grew. The strong Warburg position never disappeared and never recovered to majority. It bumps along at single-digit percentages across decades, which suggests it functions as a minority position that keeps the field honest rather than a view that is gaining ground.

## What it may mean

A few things, with appropriate hedging.

Warburg's *observation* is settled. PET imaging exists because tumors really do gulp glucose, and oncologists use that fact every day to find lesions. The clinic does not care whether the gulping is cause or consequence. It cares that tumors gulp.

The strong causal claim, as stated by Warburg himself, looks wrong. About 8% of the modern field defends it, which is not nothing, but it is also not where the evidence has gone. The dominant picture is that oncogenic activation comes first and remodels metabolism downstream. That picture is not airtight (transformation experiments using only metabolic perturbations are rare and contested), but it is where most of the data sits.

The intermediate position, that specific metabolic enzymes can be oncogenic drivers in specific tumor types, looks vindicated. Vorasidenib, an IDH1/IDH2 inhibitor, was FDA-approved in August 2024 for grade 2 IDH-mutant glioma after the INDIGO trial. It works by blocking the production of 2-hydroxyglutarate. This is a direct therapeutic exploitation of the claim that metabolism, in these tumors, is the driver. That is not Warburg's exact claim, but it is in the same neighborhood, and it took roughly a century to get there.

I would not call any of this a clean vindication. The ketogenic-diet trials remain inconclusive. 2-deoxyglucose, 3-bromopyruvate, and dichloroacetate have had uneven results. Vorasidenib is a real win, but it is one drug, in one tumor class, with a specific mechanism that is narrower than what Warburg proposed.

## Loose ends

The IDH-mutant story deserves a piece of its own. It is the cleanest example of metabolism as a causal driver, and it would be useful to count, in the same way, how many of the 12 IDH papers in this sample explicitly cite Warburg, and how many newer onco-metabolites (D-2HG, fumarate, succinate from FH/SDH-mutant tumors) are creeping into the literature. With another week I would also try to replicate the stance classification with a different model, ideally one that is not in the same family as GPT-4o-mini, to see how much of the 8.3% number is real and how much is one classifier's habit.

If anyone has a curated dataset of papers that explicitly defend the strong Warburg position post-2010, please get in touch. The 34 abstracts in that category are not enough to map the lineage properly, and I suspect there is an interesting sociological story about who keeps that flame and why.
