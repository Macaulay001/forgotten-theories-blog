# I tested Propp's 1928 folktale structure on 212 tales from 7 cultures. He may have been right.

*Part 8 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In 1928, in Leningrad, a young folklorist named Vladimir Propp published a slim book called *Morphology of the Folktale*. He had read his way through one hundred Russian fairy tales from the Afanasyev collection and made a strong, almost suspicious claim: every tale, he wrote, is built from the same 31 narrative functions, and those functions appear in a fixed order. Not every function shows up in every tale, but when they do, they line up like beads on a wire.

The book did not have a comfortable life. Formalism was politically unwelcome in the USSR and Propp's project was pushed aside. An English translation only appeared in 1958. Lévi-Strauss critiqued it shortly after. By the time post-structuralism took over the humanities, structuralism in this hard, taxonomic mode was widely treated as a dead end. Propp influenced Greimas and a chunk of Joseph Campbell's monomyth, but the original claim was never, as far as I can tell, tested at scale against folktales from outside the Russian corpus he built it from.

That seemed like a thing worth checking.

## What I tried

The plan was small. Get a folktale corpus from more than one tradition. Ask a language model to identify which of Propp's 31 functions appear in each tale, in order. Look at the result.

For the corpus I pulled 212 tales from Project Gutenberg covering seven traditions. The Grimm brothers for German. Hans Christian Andersen for Danish. Afanasyev for Russian (which is the original ground Propp built on, so its tales serve as a sanity check rather than an independent test). And then four of Andrew Lang's color fairy books, the Blue, Red, Yellow, and Olive, which pull from Arabic, Persian, Indian, African, Japanese, and various European sources. Lang was a sloppy curator but a wide one, which is what I wanted.

I used GPT-4o-mini to classify each tale. The prompt was direct: here are Propp's 31 functions with one-line descriptions, here is the tale, return JSON with the list of function labels in narrative order and any extras the model thinks fall outside the 31. One call per tale, around 1500 to 2000 input tokens and a couple hundred output. The whole sweep cost about five cents.

The classifier core looked like this:

```python
PROMPT = """Identify which of Propp's 31 functions appear in this tale, in the
order they appear. If something doesn't fit any function, note it as EXTRA.
Output strict JSON: {"functions": ["I", "VIII", "XI", ...], "extras": [...]}.

Functions:
{function_list}

Tale: {title}
{tale_text}
"""

for tale in tales:
    r = call([{"role": "user", "content": PROMPT.format(...)}],
             model="openai/gpt-4o-mini", max_tokens=400)
    d = json.loads(r["text"])
    out.write(json.dumps({"title": tale["title"],
                          "functions": d["functions"],
                          "extras": d["extras"]}) + "\n")
```

The two tests I cared about were Propp's ordering claim and his closure claim. For ordering: take each classified tale, compare the observed sequence of function labels against Propp's predicted positions, compute a Spearman rank correlation. For closure: count how many "extras" the model reports that genuinely fall outside the 31, versus how many are just classifier noise like "this passage is a preface".

If Propp was wrong about either, this should be visible. If the ordering was loose, ρ would scatter near zero. If the function set was open, the extras column would fill up with new categories.

## What happened

The numbers came out further on Propp's side than I expected.

Across 190 tales where the model identified enough functions to compute a rank correlation, the mean Spearman ρ between Propp's predicted order and the observed order was 0.938. Ninety-eight percent of those tales had ρ above 0.5. The distribution is not flat at zero, not centered at 0.5, it sits up near 1.

For closure, the LLM proposed 12 unique "extras" across 212 tales. Most of them were classifier errors, things like "the text begins with a preface" or "the narrator addresses the reader". After filtering those out, only two extras looked like real plot beats that did not fit any of Propp's 31: a "boasting object" function (a magical item that announces its own powers) and "learning the language of birds" as a discrete step rather than as part of acquisition. Two. In a corpus of 212 tales from seven traditions.

The function frequency table is where I think the most interesting thing lives. A small set of functions dominates everything.

| Function | Share of tales |
|---|---|
| VIII Villainy or Lack | 87% |
| IX Mediation | 91% |
| X Counteraction | 93% |
| XI Departure | 93% |
| XX Return | 70% |
| XIV Acquisition | 67% |
| XII Test by donor | 48% |
| XVI Struggle | 43% |
| XVIII Victory | 46% |
| XXXI Wedding | 35% |
| XVII Branding | 0% |
| XXIV–XXVI False claim sequence | 2–3% |

The kernel that almost every tale shares is four steps: something is wrong (villainy or lack), someone tells the hero (mediation), the hero agrees to act (counteraction), the hero leaves home (departure). Roughly 85 to 93 percent of tales in the corpus hit all four. Whatever a folktale is, structurally, in this sample, it is at minimum that.

The tail is more uneven. Branding does not appear once. The false-claim sequence, where a pretender hero claims the win and is later exposed, is rare in Lang and Andersen and Grimm. Those may be more Russian-specific or Afanasyev-specific. I would not lean hard on that without checking more carefully.

I had a moment of doubt about the ordering number. A Spearman ρ of 0.94 is high enough to be suspicious. So I looked at the tales with the lowest ρ values. They were mostly short Andersen pieces that are arguably not folktales at all, things closer to moral parables or vignettes, where the function sequence is partial and short enough that two transpositions tank the correlation. Removing very short tales (under five identified functions) pushed the mean up further. The ordering signal is not driven by a long tail of low-ρ outliers being averaged with high-ρ ones, it is broadly distributed near the top.

## What it may mean

I want to be careful here. This is one experiment, with one classifier, on one corpus pulled mostly from English-language anthologies of varying editorial quality. It does not establish that Propp was right about folktales in general. It does establish, I think, that his specific empirical claim has the kind of shape that survives a first serious cross-cultural pass.

The structural skeleton, villainy then mediation then counteraction then departure, looks like a real cross-cultural pattern in this sample. Whether that is because the tales are descended from a common Indo-European story stock, or because human narrative cognition reaches for this skeleton independently, or because Lang and Grimm and Afanasyev's editors quietly shaped their material toward a familiar Western shape, I cannot tell from this data.

What I think is worth saying out loud is that Propp's framework, which has been treated for decades as a quaint formalist exercise, makes a falsifiable prediction about narrative structure, and the prediction survives a cheap modern test. That is a small thing but not nothing. Structuralism in its strong taxonomic mode is supposed to be dead. On this evidence, in the narrow domain Propp claimed for it, it does not look dead.

This is a modest discovery, not a vindication. The contribution is the cross-cultural quantification at scale, not the qualitative claim, which Propp already made.

## Loose ends

The largest hole is also the most obvious. GPT-4o-mini has almost certainly read Propp during training, and it has read summaries of his 31 functions and probably some Proppian analyses of specific tales. When I ask it to label tales by Propp's functions, I am not running a clean test of whether the structure is there, I am running a test of whether the model can find the structure that Propp told it to find. The cleaner experiment would use a never-seen, non-Propp narrative framework as a control, classify the same tales, and check whether that other framework also yields a ρ of 0.9. If it does, the result is mostly about the classifier. If it does not, Propp's claim looks stronger. That is the next thing I want to do, and it costs under a dollar.
