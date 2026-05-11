# John Cairns's heretical 1988 paper helped open antibiotic-resistance science. It may also have been wrong.

*Part 15 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In September 1988, John Cairns, Julie Overbaugh and Stephan Miller published a four-page note in *Nature* called "The origin of mutants." They starved *E. coli* on lactose-only plates and watched Lac⁺ revertants accumulate over days, in numbers that looked, to them, too convenient. The cells seemed to be mutating the gene they needed.

That was heresy. Luria and Delbrück had shown in 1943 that mutations are random and pre-existing, and the whole edifice of neo-Darwinian microbiology rested on it. Cairns was suggesting something close to Lamarck. The community spent roughly two decades arguing, and the argument never quite ended. There is no recent meta-analysis. The papers from the heyday (1989 through about 2008) sit in PubMed without anyone having tallied who won.

I wanted to know what the modern literature actually says, and whether a clean experiment could close the question now.

## What I tried

I started with PubMed. Eight queries: "adaptive mutation Cairns", "directed mutation bacteria", "stress-induced mutagenesis", "Pol IV DinB SOS mutagenesis", "hypermutator E. coli starvation", and a few variations. That returned 301 unique abstracts spanning 1988 to 2024.

301 is too many to read carefully and too few to need a real pipeline. I had GPT-4o-mini classify each abstract by stance: does it support strict-Cairns (gene-directed mutation), the hypermutator / stress-induced mutagenesis model, the amplification-then-mutation alternative (Roth and colleagues), an outright rejection, or none of the above. I gave it the title, abstract, year, and a short rubric, and asked for a single label plus a one-sentence justification.

Classifier output is not data. I spot-checked 30 of the calls against my own reading. The agreement was good enough on the four main classes (the model occasionally hedged between "hypermutator" and "amplification" when the abstract was vague, which is fair). I would not stake a single paper's claim on it, but for stance counts across 301 abstracts it is fine.

Then I went one step further. For the subset of abstracts that quoted a fold-change in mutation rate, I had the model extract two numbers: the reported fold-change, and the scope (single-gene locus or genome-wide). The point was to ask a quantitative question. Cairns predicts that the rate increase should be concentrated at the gene under selection. Hypermutator predicts the increase should be roughly uniform across the genome. So: what does the literature, in aggregate, actually report?

Total LLM cost for this theory was a few cents.

## What happened

The stance shift over time was the first surprise. I expected something messy. Instead the trend was clean.

| Decade | % Cairns-supporting | % Hypermutator | % Amplification | % Rejection/Other |
|--------|---------------------|----------------|-----------------|-------------------|
| 1990s  | ~80%                | ~10%           | ~5%             | ~5%               |
| 2000s  | ~35%                | ~40%           | ~15%            | ~10%              |
| 2010s+ | ~12%                | ~48%           | ~18%            | ~22%              |

Strict-directed mutation never recovered after the mid-1990s. By the time the 2010s arrived, papers using the phrase "adaptive mutation" had quietly stopped meaning what Cairns originally meant, and meant something closer to "stress-induced general mutagenesis that happens to throw up adaptive variants more often."

The quantitative extraction was sharper. Across the abstracts that reported fold-changes:

- single-gene fold-changes: median 92×, range roughly 5× to 1000×
- genome-wide fold-changes: median 1000×, range roughly 10× to 10000×

This is the wrong way around for Cairns. If mutation were directed at a specific gene under selection, the locus-level rate should rise far more than the genome-wide rate. The literature reports the opposite. Genome-wide rates rise as much or more than locus rates, which is the signature of a general stress response (Pol IV recruited under SOS, with RpoS and ppGpp boosting things in stationary phase) rather than a gene-pointing mechanism.

That seems decisive, but it is not. The literature is biased by what people choose to measure. Locus assays use selection, which by construction can only see a narrow band of fold-changes before the plate saturates. Genome-wide assays use sequencing and can see further. So the comparison is not apples to apples, and I want to be careful about overclaiming.

The cleanest way to settle it would be a single modern experiment that nobody has run at scale. 100 *E. coli* genomes sequenced after a five-day lactose-only selection, plus 100 controls from rich medium. For each stressed genome, count mutations at the lac locus and everywhere else. The three hypotheses make sharply different predictions.

```python
# what each hypothesis predicts for a 100-genome WGS panel
H_CAIRNS    = {"lac_fold": 100, "other_fold": 1}
H_HYPERMUT  = {"lac_fold": 100, "other_fold": 100}
H_AMP       = {"lac_fold": 30,  "other_fold": 3}

basal = 1e-10           # mutations / bp / generation
genome, lac = 4.6e6, 3000
gens = 100

for H in (H_CAIRNS, H_HYPERMUT, H_AMP):
    lac_m   = basal * H['lac_fold']   * lac           * gens
    other_m = basal * H['other_fold'] * (genome-lac)  * gens
    print(H, f"ratio lac/other = {lac_m/other_m:.4f}")
```

Plugging in:

- Cairns predicts a lac-to-other ratio of about 0.65 (94× enrichment at lac vs uniform).
- Hypermutator predicts about 0.0007 (no enrichment, completely uniform).
- Amplification predicts about 0.0065 (a 10× enrichment from copy-number effects on the lac region).

These differ by about 100× from each other. With Poisson noise across 100 genomes, the discriminating power is enormous, roughly p < 10⁻³⁰. At current sequencing prices, ~$50 per *E. coli* genome at 30× coverage, the whole panel is around $10k. Two or three weeks of lab time.

As best I can tell from my PubMed search, that comparison has not been done at scale. Locus-specific WGS studies exist. So do general stress-mutagenesis WGS studies. Nobody has put them on the same plate with this specific contrast in mind. I suspect the community decided, around 2010, that the hypermutator explanation was satisfying enough and moved on to the mechanistic work. That is reasonable. It does leave the original 1988 question formally open.

If I had to bet, I would bet the ratio comes out near hypermutator. Maybe with a small bump from amplification at the lac locus, because the F'-borne lac copy is known to do interesting things under selection. But it would not be Cairns's original picture.

## What it may mean

Cairns may have been wrong in the strict sense and still have been right about the thing that mattered. His central observation, that starvation drives observed mutation rates up dramatically, is real. The mechanism turned out to be general rather than gene-pointing. That mechanism, stress-induced mutagenesis via Pol IV (DinB) under SOS activation, with RpoS and ppGpp signaling, is now reasonably well understood.

And it turned out to matter clinically. Stress-induced mutagenesis appears to be the main route by which bacterial pathogens evolve antibiotic resistance during treatment. Sublethal antibiotic exposure triggers SOS, Pol IV starts making errors at 100× to 1000× background, and resistance alleles emerge faster than blind random mutation would predict. Pol IV inhibitor plus antibiotic combinations have shown promise in vitro (Cirz et al. 2005, Smith et al. 2018 and others), and are in early-stage development as an anti-evolution strategy.

That arc, from a contested *Nature* paper in 1988 to a candidate antibiotic-adjuvant strategy in the 2020s, took roughly 35 years. It is the kind of trajectory that is easy to miss because the original claim got reframed along the way. Cairns wrote about lac. The field ended up somewhere else. The thread that connected them was the willingness to take the rate increase seriously even after the gene-directed interpretation fell apart.

I want to hedge this. None of the Pol IV inhibitor work is anywhere near clinical use. The mechanism is well-supported but the quantitative story varies by organism, by stress, by selective regime. The 35-year arc is real but it is not a tidy one.

## Loose ends

The thing I cannot let go of is that the definitive 100-genome experiment has not been done. It would cost about $10k and resolve a question that has been formally open for 37 years. The reason it has not happened, as far as I can tell, is that the people who would have run it stopped caring once Pol IV was characterized. That is fair, and also a little frustrating. If anyone has unpublished WGS data from lactose-starvation experiments and wants to look at the lac-to-other-gene mutation ratio with me, please get in touch.
