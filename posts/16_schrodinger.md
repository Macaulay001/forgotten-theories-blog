# Schrödinger predicted DNA in 1944. He may also have predicted its 4% inefficiency.

*Part 16 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In February 1943, Erwin Schrödinger gave a set of public lectures at Trinity College, Dublin, that he later wrote up as the slim Cambridge book *What is Life?* (1944). He was a physicist trying to think his way into biology, and he made three guesses that should not have been possible at the time. He guessed that the hereditary substance is an "aperiodic crystal", a covalent solid that carries information precisely because it is not regular. He guessed that life sustains itself by feeding on "negative entropy", importing order from its environment. And he guessed that mutations are quantum-discrete events, isomeric jumps inside molecules.

The first guess was vindicated in 1953 by Watson and Crick, who quietly credited the book in interviews. The third was partly captured by Löwdin's proton-tunneling work in 1963. The middle one, the negative-entropy claim, has always felt the most slippery. I wanted to see whether you could make it numerical.

## What I tried

The version of Schrödinger's claim that you can actually test is this: if DNA is a genuine aperiodic crystal carrying biological information, its per-base Shannon entropy should sit close to the theoretical 2-bit ceiling, but not at it. A perfectly random sequence has 2 bits per base. A periodic sequence (AAAAAA, or any repeat) has 0. Real biological information should lie in between, and probably nearer the top than the bottom.

I pulled seven whole-genome assemblies from NCBI that span most of the tree of life: *E. coli* (a clean bacterial genome, ~4.6 Mb), *S. cerevisiae* (yeast, fungi), *C. elegans* (a small animal genome, ~100 Mb), *Drosophila melanogaster* (insect), *Arabidopsis thaliana* (plant), *Haloferax volcanii* (a halophilic archaeon), and human chromosome 21 as a vertebrate sample (~45 Mb, the smallest human autosome and a convenient stand-in). The script that fetched and unpacked them is small. For each genome I computed k-mer Shannon entropy for k=1 through k=6, in bits per base, on a single concatenated sequence with Ns stripped.

The whole pipeline ran in under ten minutes on a laptop. Here is the core of the entropy calculation, which is really not more than a Counter and a sum.

```python
def kmer_entropy(seq, k):
    counter = Counter()
    for i in range(len(seq) - k + 1):
        kmer = seq[i:i+k]
        if "N" in kmer: continue
        counter[kmer] += 1
    total = sum(counter.values())
    H = 0.0
    for c in counter.values():
        p = c / total
        H -= p * math.log2(p)
    return H  # bits per k-mer; divide by k for bits/base
```

The plan after that was straightforward. Phase 2 gave me whole-genome numbers. Phase 3 broke each genome into 1-kb windows and looked at the distribution of local entropies. Phase 4 tried to attribute the gap below 2 bits to specific contributions: GC bias (mononucleotide), dinucleotide bias on top of that, trinucleotide bias on top of that, and so on.

## What happened

Every genome I looked at sat between 1.86 and 1.96 bits per base at k=6. That is 93 to 98 percent of the Shannon ceiling. *E. coli* came in at 1.963 bits per base, the highest in the set. Human chromosome 21 came in at 1.924. Yeast was 1.946. *Haloferax volcanii*, an extreme halophile with a strongly GC-skewed genome, was the lowest at 1.862. So on this sample, Schrödinger's aperiodic-crystal claim is straightforwardly correct as a quantitative statement: real DNA is much closer to "random" than to "periodic", and the gap to 2 bits is small.

What surprised me was the structure of the gap.

The sliding-window analysis was the first hint. For *E. coli*, 61% of all 1-kb windows have entropy above 1.95 bits per base. The genome is, in an information-theoretic sense, nearly uniform across its length. For human chromosome 21, only 1.6% of 1-kb windows clear the same threshold. The vertebrate genome is not less dense on average by a huge margin (1.92 versus 1.96, a 2% gap), but locally it is full of windows that drop well below the bacterial baseline. Repetitive elements (transposons, simple repeats, satellite-like sequences) account for some of this, but they are under half a percent of windows in every genome I checked, so they cannot explain the bulk of the difference on their own.

Phase 4 is where the story got specific. I decomposed each genome's total deficit (2.0 minus its bits/base) into contributions from successively higher-order correlations. The mononucleotide term is GC bias. The dimer term is what you get from over- or under-represented dinucleotides on top of whatever GC bias already explains. The trimer term is the next residual.

| Organism      | Total deficit | GC bias | Dimer corr | Trimer corr |
|---------------|--------------:|--------:|-----------:|------------:|
| E. coli       | 0.037         | 0.000   | 0.018      | 0.018       |
| Yeast         | 0.054         | 0.041   | 0.010      | 0.004       |
| Drosophila    | 0.042         | 0.019   | 0.017      | 0.006       |
| Human chr21   | 0.076         | 0.024   | 0.040      | 0.012       |
| Arabidopsis   | 0.076         | 0.058   | 0.013      | 0.005       |
| Haloferax     | 0.138         | 0.070   | 0.058      | 0.010       |
| C. elegans    | 0.103         | 0.058   | 0.036      | 0.009       |

Three patterns showed up.

The *E. coli* deficit is almost evenly split between dimer and trimer contributions, with effectively zero GC bias. That is what you would expect from a genome that has been compressed by selection: about half of its small departure from Shannon-random comes from dinucleotide context (the constraints of being a coding-dense bacterial chromosome with strong codon-context selection), and the other half from trinucleotides (codon usage proper). It is the most "Shannon-efficient" genome in the set.

Human chromosome 21 has a different fingerprint. Its dimer term, 0.040 bits per base, is the largest in the table apart from *Haloferax*, and it dwarfs every other contribution in the same row. That is almost certainly CpG depletion. Vertebrate genomes methylate CpG dinucleotides at the cytosine. Methylcytosine deaminates to thymine roughly an order of magnitude faster than ordinary cytosine, and over evolutionary time the result is a global under-representation of CpG. The information-theoretic shadow of that biochemistry is sitting right there in the dimer column. I did not need to assume CpG depletion to find it; I just decomposed the entropy deficit by order, and the dimer term lit up.

*Haloferax* is the most informationally "lossy" of the seven, and most of that comes from extreme GC content (around 65 to 67%). That is a mononucleotide effect that almost any halophile shows, presumably tied to thermal and osmotic stability of GC-rich DNA in high-salt environments. *Arabidopsis* and *C. elegans* sit somewhere in between, with their deficits dominated by GC bias rather than higher-order correlations.

So a single number (bits per base) hides a story that you can pull apart into a few biologically interpretable terms. The 4% gap below the Shannon ceiling in human chromosome 21 is not noise. About half of it is one specific evolutionary process (methylation-driven CpG depletion), and the rest is a mix of GC bias and codon-context effects.

## What it may mean

If this generalises, and that is a real if (seven genomes, one human chromosome rather than the whole assembly, a single k=6 entropy estimator), Schrödinger's aperiodic-crystal hypothesis looks better than it had any right to look. The hereditary substance is, in fact, an aperiodic crystal that sits within a few percent of the Shannon limit, and the small gap is not random. It tracks identifiable biological mechanisms: codon usage, methylation chemistry, base-composition pressure from environment.

The negative-entropy framing then has a concrete reading. The "bits of order" a kilobase of human DNA imports from its environment, relative to a random control, is about 50 bits (0.05 bits per base times 1000). That is small in joules at 300 K, on the order of 10^-22, which is one reason the negative-entropy claim has always felt suspicious in pure thermodynamic terms. The accounting that matters is in bits, not in calories.

The practical reading is for synthetic DNA storage. Microsoft, Catalog, and Twist Bioscience are storing data in synthesised oligonucleotides at roughly 1.6 bits per base in deployed systems. The theoretical ceiling is 2.0. Real biological DNA sits at 1.86 to 1.96. There is real headroom between the synthetic and biological numbers, somewhere between 0.25 and 0.35 bits per base, which is not a tiny fraction. Some of that headroom is unavoidable (error-correcting overhead in synthesis), but not all of it. Biology is operating closer to the ceiling than current engineering does.

I should hedge the obvious things. Shannon k-mer entropy is a scalar. It captures local statistics out to length k. It does not see long-range correlations, gene structure, or three-dimensional chromatin organisation, any of which might carry additional information at scales the estimator is blind to. And "vindicated" is doing a lot of work in a claim built on seven genomes.

## Loose ends

The next thing I would do is run a Lempel-Ziv complexity estimate on the same windows. LZ catches long-range redundancy that k-mer Shannon misses, so the gap between the two estimators is informative on its own. I would also like to redo the sliding-window analysis stratified by annotation (CDS, intron, intergenic, repeat) rather than as a single histogram, because the 1.6% versus 61% comparison between human and *E. coli* is interesting but probably explained mostly by the coding fraction. If anyone has clean annotations for the seven genomes in a single comparable format, I would like to talk. The whole experiment cost zero dollars and about fifty minutes of compute.
