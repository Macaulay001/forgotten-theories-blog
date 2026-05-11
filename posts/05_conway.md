# Melvin Conway said software design mirrors org structure. In 19 GitHub repos, 16 say he's mostly right.

*Part 5 of **Forgotten Theories**, a series re-testing old scientific claims with modern tools. Find the rest under the `#forgotten-theories` tag.*

---

In April 1968, Melvin Conway published a short paper in *Datamation* called "How Do Committees Invent?" (volume 14, issue 5, pages 28 to 31). The piece argued one specific thing: any organization that designs a system will produce a design whose structure copies the organization's communication structure. People who built different parts of the product didn't talk much across the gap, so the gap shows up in the code.

The line got famous as "Conway's Law", but mostly as a slogan. MacCormack, Rusnak and Baldwin tested it on a handful of commercial products in 2006 and 2012 and found supporting evidence, but I couldn't find a wider panel study. So it sits in this odd place where everyone quotes it and almost nobody has run the test on more than five projects.

I wanted to see what happens on 19 mature open-source repositories where the entire commit history is sitting on GitHub waiting for anyone to pull it.

## What I tried

The plan was straightforward in shape and a bit awkward in execution. For each repo, clone the last 5000 commits with `--filter=blob:none --no-checkout` so I wouldn't have to download blobs I didn't need, then walk the last 3000 non-merge commits and record, for every file changed, who changed it. From that I get two partitions of the file set.

The first partition is the **top-directory** label for each file. If the file is `src/transport/http2.c`, the label is `src`. This is a rough proxy for "which module of the system does this file belong to". It's crude, and I'll come back to that.

The second partition is the **top contributor** for each file, restricted to authors with at least 5 commits in the window. If a file has been touched 30 times and the same email account is responsible for 18 of those, that account becomes the file's label. This is a proxy for "which person mainly owns this file", and through that, "which informal sub-team" if you squint.

Conway's claim, translated to these two partitions, says they should agree more than chance. The people who mainly own `src/transport/*` should not be the same people who mainly own `src/crypto/*`. If they were the same people, the org would be one undifferentiated blob and Conway's prediction would fail.

To measure agreement I used Normalized Mutual Information between the two partitions on the same file set. NMI is between 0 and 1. Higher means the two labelings carry more information about each other.

The null model is the part I cared most about getting right. I shuffled the commit-to-author assignments 20 times, keeping the total number of commits each author made fixed, then recomputed the file-to-top-contributor partition under each shuffle. This destroys real specialization (who works on what) while preserving the contribution-volume distribution (the long tail of one-off committers stays a long tail). NMI under shuffle gives me what to expect from pure clustering math, with no Conway-style ownership at all.

The 19 repos: openssl, numpy, pandas, scikit-learn, django, rails, redis, git, curl, grafana, flask, sympy, networkx, scipy, matplotlib, requests, notebook, keras, yt-dlp, click. I picked a mix of languages and project ages so the result wouldn't be a Python-monoculture artifact.

## What happened

16 of 19 projects had real NMI above shuffled NMI. Mean NMI(real) = 0.236, mean NMI(shuffle) = 0.187. The real/shuffle ratio averaged around 1.26x, so the lift is roughly 5 percentage points of mutual information above chance.

This is the core of the test, written without imports or boilerplate:

```python
# for each file: top directory and top contributor (>= 5 commits)
for c in commits:
    for f in c["files"]:
        file_dir[f] = f.split("/", 1)[0] if "/" in f else "_root"
        file_author_counts[f][c["author"]] += 1

for f, ac in file_author_counts.items():
    relevant = {a: n for a, n in ac.items() if a in common_authors}
    file_top_author[f] = max(relevant, key=relevant.get) if relevant else "_singleton"

labels_dir  = [file_dir[f]        for f in files]
labels_auth = [file_top_author[f] for f in files]
nmi_real = normalized_mutual_information(labels_dir, labels_auth)
# then 20 shuffles of commit -> author, recompute NMI each time
```

The interesting structure is in which repos behave which way. Sorted by z-score (real NMI standardized against the shuffle distribution):

| repo     | NMI(real) | NMI(shuf) | z    |
|----------|-----------|-----------|------|
| git      | 0.276     | 0.184     | 8.2  |
| rails    | 0.208     | 0.151     | 7.4  |
| scipy    | 0.213     | 0.145     | 4.4  |
| requests | 0.412     | 0.278     | 3.7  |
| click    | 0.252     | 0.191     | 2.7  |
| curl     | 0.212     | 0.117     | 2.2  |
| keras    | 0.133     | 0.133     | 0.0  |
| networkx | 0.131     | 0.130     | 0.0  |
| sympy    | 0.300     | 0.339     | -0.9 |
| notebook | 0.216     | 0.281     | -1.7 |

git is the strongest Conway-mirror by a long way. That tracks with how git is actually maintained: there are subsystem maintainers who own specific top-level directories (`builtin/`, `t/`, `Documentation/`, `contrib/`) and most of their commits land in those directories. Rails is similar, with the activerecord, actionpack, activesupport gem boundaries doing roughly what Conway predicted social boundaries should do.

The weakest cases are also informative. keras and networkx show almost no lift. sympy and notebook actually have real NMI *below* the shuffle mean, which I did not expect. Looking at the commit logs, these are all projects where one or two maintainers commit very broadly across the entire tree, plus a long tail of one-off contributors who each touch a small number of files in different corners. Under the shuffle null, those one-off contributors get scattered uniformly, which paradoxically *increases* apparent clustering of files by author. The shuffle is doing more "Conway work" than the real assignment, because the real assignment has a small set of people pinging everywhere.

So the test isn't picking up something pristine. It seems to pick up the difference between "many semi-specialists" (git, rails, scipy) and "few generalists plus drive-by patches" (keras, sympy, notebook). Conway's claim seems to hold cleanly in the first regime and breaks down in the second.

I should also flag that I'm running 19 tests in parallel without multiple-comparison correction. With a one-tailed test the binomial probability of getting 16/19 above shuffle by chance, with no Conway effect at all, is around 0.002, so the panel-level claim is fine. The per-repo z-scores should be read more cautiously.

## What it may mean

Taken at face value, this suggests Conway was directionally right in 1968 and that the effect is real but modest at the level of open-source repositories. A 5-percentage-point NMI lift over a degree-preserving null isn't dramatic. It isn't nothing either.

The result lines up with the MacCormack 2006 finding that product modularity correlates with org modularity, but at larger scale (19 projects instead of a handful) and with a stricter null (preserving contribution volume rather than free permutation). It also suggests an extension Conway didn't write about: the law may apply most cleanly in projects that have grown a sub-team structure organically. When one or two people maintain the whole thing, there's no informal social boundary to mirror, so there's nothing for the code's module boundaries to copy.

I don't want to claim too much from this. The directory-name proxy could be partially circular if maintainers literally rename directories to match their own areas, though I doubt this drives most of the signal because the top directories in most of these repos predate the current maintainers by years. I also can't rule out that part of the NMI lift is just file-locality, the fact that someone fixing a bug in one file tends to touch nearby files in the same commit. That would inflate the real NMI without any Conway-style ownership, and my shuffle null doesn't fully control for it.

## Loose ends

Things I'd do with another week. Use a proper module graph from co-change frequency instead of top-directory name, which would address the locality concern. Try the test on Linux kernel subsystems, where the social structure is documented in MAINTAINERS and you could compare against ground truth rather than a proxy. Restrict to authors with at least 50 commits to remove the long-tail effect that seems to be flipping the sign on keras and notebook. The whole pipeline runs in about 40 minutes on a laptop if you have the bandwidth to clone 19 repos, and the script is one file. Anyone with patience and a decent internet connection can replicate or break this in an afternoon.
