# I fit Zipf's 1949 law on four books in three languages. The slope came out 1.05.

*Part 28 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1949, the Harvard linguist George Kingsley Zipf published a strange, sprawling book called *Human Behavior and the Principle of Least Effort*. It was part linguistics, part economics, part anthropology, and partly a theory of why cities, incomes, marriages, and word frequencies all seemed to obey the same arithmetic. The piece that survived best is the simplest. Rank the words of any large text from most to least common. Call rank 1 the most frequent word, rank 2 the next, and so on. Then the frequency of the word at rank r falls roughly as 1/r. Plot it on log-log axes and you get a straight line with slope about negative one.

Zipf checked this on English, Latin, Plautine drama, Chinese, and (in a chapter that has aged poorly) the speech of psychiatric patients. The "principle of least effort" was his catch-all mechanism: speakers want to be lazy and reuse words, listeners want disambiguation, the equilibrium produces the power law. The mechanism is contested. The pattern, less so. But I had never actually fit it myself, and the original numbers come from a world before computers.

So I spent an evening doing it on four books.

## What I tried

I pulled four Project Gutenberg texts: Moby Dick (English, 1851), the standard English translation of War and Peace (Tolstoy, originally 1869), Don Quixote in Spanish (Cervantes, 1605), and Tome I of Les Misérables in French (Hugo, 1862). Token counts ranged from about 112,000 (the Hugo volume) to about 573,000 (Tolstoy). Vocabularies sat between 12,899 and 23,132 distinct word forms.

Tokenization was deliberately crude. A Unicode-letter regex with apostrophes, lowercase. No lemmatization, no enclitic splitting. That means French `du` gets counted on its own and Spanish `del` likewise, which would horrify a corpus linguist, but for a rank-frequency plot it changes the answer in the third decimal at most.

For each book I built the rank-frequency table and fit two ways. The first is what Zipf actually drew: ordinary least squares on the log-log plot of the top 1000 ranks. That's a biased estimator (everyone since Newman 2005 has said so), but it's the historical comparison. The second is the Clauset, Shalizi, and Newman (2009) maximum-likelihood method applied to the count distribution, with the lower cutoff x_min picked by minimizing the Kolmogorov-Smirnov distance between empirical and fitted CDFs. The MLE returns the exponent of the *count* distribution, which corresponds to the rank exponent via a simple change of variables.

Roughly:

```python
ranks, counts = rank_freq(tokens)          # sorted descending
slope, intercept = np.polyfit(             # Zipf's own plot
    np.log(ranks[:1000]),
    np.log(counts[:1000]), 1)
alpha_zipf = -slope

for x_min in candidate_cutoffs:            # CSN style
    alpha = mle_discrete_powerlaw(counts, x_min)
    ks = ks_distance(counts, x_min, alpha)
    if ks < best_ks: best = (alpha, x_min, ks)
```

That's the entire scientific content. Everything else is bookkeeping.

## What happened

The four OLS slopes came in remarkably tight:

| Corpus | Tokens | Vocab | OLS α (top 1000) | MLE α (counts) | implied α_rank | KS |
|---|---:|---:|---:|---:|---:|---:|
| Moby Dick (EN) | 219,035 | 16,957 | 1.07 | 1.94 | 1.06 | 0.085 |
| War and Peace (EN) | 573,084 | 17,484 | 1.05 | 1.80 | 1.25 | 0.073 |
| Don Quixote (ES) | 383,717 | 23,132 | 1.05 | 1.85 | 1.17 | 0.081 |
| Les Misérables I (FR) | 111,887 | 12,899 | 1.04 | 1.93 | 1.08 | 0.097 |

Mean OLS α across the four texts is 1.05 with a spread of ±0.02. Zipf wanted 1.0. He got 1.05 to two decimals on books he never saw, in languages he never specifically tested, with a method he didn't have. The original claim survives.

The MLE on the count distribution is a different beast. It returns exponents in the 1.80 to 1.94 range, which sounds wrong until you remember that this is the exponent of P(f), not f(r). Converting back gives implied rank exponents between 1.06 and 1.25, the upper end of which (Russian-translated-to-English and Spanish) hints at a steeper-than-1 slope. That matches what Ferrer-i-Cancho and Solé (2001) called the two-regime structure: shallow slope for the common-word head, steeper for the rare-word body. Either method gives a power law that fits within KS ≤ 0.10.

Hapax legomena (words appearing exactly once) are the part of every Zipf plot that flattens or bends. They were 33% of the Tolstoy vocabulary, 43% of Melville's, 49% of Cervantes's, and 54% of the Hugo volume. The more inflected the language and the smaller the corpus, the higher the hapax fraction. That part of the curve isn't a power law and never was. The Clauset method explicitly throws those low-frequency points away by picking x_min around 9 to 13.

One detail caught me by surprise. The slope barely cares about language, but it does care about genre. I didn't test this here, but if you take Twitter dumps or news headlines the slope shifts noticeably (people have reported α closer to 0.9 for telegraphic text and around 1.2 for technical prose). Books written for a literate adult reader from any of three Indo-European languages, by contrast, gave essentially the same number.

The "implied α_rank" column has a small puzzle in it. The two corpora with the highest implied exponents are War and Peace (1.25) and Don Quixote (1.17). The Hugo and Melville texts come in at 1.08 and 1.06. I am not confident this is signal rather than noise from MLE on a heavy tail, but if it is signal, the candidates are translation artifacts (the Tolstoy is a translation) and a richer Spanish morphology that spreads frequency across more word forms. With four texts and one tokenizer, I can't tell those apart.

What does feel real is that the head of every curve, the first 30 to 50 most common words, is fit by a slope of essentially negative one. *The* and *of* and *and* in Moby Dick; *que* and *de* and *y* in Don Quixote; *de* and *la* and *et* in Hugo. The arithmetic linking their frequencies is the same to a fraction of a decimal place.

## What it may mean

Zipf's original explanation, the principle of least effort, is hard to formalize. What's been clearer since Mandelbrot (1953) is that lots of generating processes produce something close to 1/r in their rank-frequency tails. Random typing on a keyboard with a space bar produces it (Miller 1957). Yule processes produce it. Preferential attachment produces it. So a tight Zipf fit doesn't tell you the words come from a mind. It tells you they come from a process with a few common things, many rare things, and some kind of multiplicative or branching structure underneath. That's a wide tent.

What may matter more for practical purposes is the deviation. Human prose deviates from Zipf in specific ways: the high-frequency head is slightly shallower than a pure power law, the tail is fatter, hapaxes are a sizeable chunk of vocabulary. Synthetic text from older n-gram models tends to flatten the head and shrink the hapax fraction. Modern LLM output is closer to human, but, on the few quick checks I've seen reported, still slightly off. A Zipf residual could be a cheap quasi-test for whether a piece of text is "natural" in the statistical sense, separate from whether it's meaningful or true.

The cross-language stability also says something about translation. The Tolstoy I used is an English version, but it still produces a roughly Zipfian curve, which suggests that translation preserves the rank structure of word use even when individual words shift. I find that mildly comforting.

## Loose ends

Four books is not a corpus study. The next step would be to run this on a few hundred Gutenberg books in a few dozen languages and look at the distribution of α values, with a proper bootstrap for confidence intervals on each. That would also let me ask whether translated texts cluster differently from native ones, and whether genre (drama, essay, novel) produces visible slope differences. The Les Misérables file I downloaded is only Tome I, about 112k tokens, which is small enough that the MLE fit is shakier than the rest. A larger Hugo corpus would tighten the French number. None of this looks expensive. You can replicate the full analysis from raw Gutenberg URLs in about ten minutes on a laptop.
