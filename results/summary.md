# Hypernymy Project — Summary of Findings (Animal Domain)

## Goal
Test whether word embeddings capture hypernymy — the "is-a" relationship
between a specific word (hyponym, e.g. "poodle") and its general category
(hypernym, e.g. "dog") — and whether the answer differs between a
**contextual** model (BERT) and a **static** one (GloVe). The animal domain
is used as a narrow, easy-to-verify test case.

## Method
1. **Dataset**: hyponym–hypernym pairs extracted from WordNet, starting at
   the `animal` synset and walking down 4 levels of its category tree.
   Result: **514 pairs**, **507 unique words/phrases**, spanning depths 0–3
   (depth = how many WordNet steps below "animal" the pair sits).
2. **Embeddings**, from two models under identical conditions:
   - **BERT** (`bert-base-uncased`): each word passed through the model on
     its own, subword token vectors mean-pooled into one 768-d vector.
     Coverage 507/507 words (100%).
   - **GloVe** (`glove-wiki-gigaword-300`): 300-d static vectors; multi-word
     phrases averaged over their constituent tokens. Coverage 409/507
     words (80.7%).
3. **Analysis**:
   - Cosine similarity of each true pair, against a random-pair control
     group, with Welch's t-test, Cohen's d, and bootstrap 95% CIs
     (1,000 resamples).
   - "Offset consistency": whether `general_vector − specific_vector` points
     in a similar direction across all pairs (the trick behind word2vec
     analogies like king − man + woman ≈ queen).
   - Similarity by hierarchical depth, tested with one-way ANOVA.
   - Qualitative case studies of each model's best, worst, and most
     disagreeing pairs.
   - 2D PCA visualization of the 507 BERT vectors, colored by depth.

## Results

### True pairs vs. random pairs

| Model | Coverage | True pairs (95% CI) | Random pairs (95% CI) | Welch's t (p) | Cohen's d |
|---|---|---|---|---|---|
| BERT (contextual) | 514/514 (100%) | **0.508** [0.495, 0.522] | 0.426 [0.415, 0.436] | 9.06 (7.3×10⁻¹⁹) | 0.565 (medium) |
| GloVe (static) | 397/514 (77.2%) | **0.367** [0.337, 0.395] | 0.103 [0.090, 0.116] | 16.15 (1.9×10⁻⁴⁸) | 1.147 (large) |

Both models separate true hypernym pairs from random pairs with high
statistical significance. GloVe shows the larger raw effect size — but see
the Interpretation section, because that result does not survive
qualitative inspection.

### Offset consistency

| Model | Mean (std) | Degenerate pairs excluded |
|---|---|---|
| BERT | 0.209 (±0.208) | — |
| GloVe | 0.224 (±0.207) | 23 (5.8% of covered pairs) |

Neither approaches 1.0: **no single consistent "hypernymy direction"**
exists in either embedding space.

### Similarity by depth (BERT)

| Depth | n | Mean | Std |
|---|---|---|---|
| 0 | 47 | 0.508 | 0.143 |
| 1 | 71 | 0.459 | 0.179 |
| 2 | 150 | 0.513 | 0.148 |
| 3 | 246 | 0.519 | 0.176 |

One-way ANOVA: **F = 2.487, p = 0.060** — narrowly short of α = 0.05, so
depth has no statistically significant effect on similarity.

### Embedding space structure
2D PCA of the BERT vectors captures only **23.7%** of total variance, but
shows loose, visible semantic clustering — mammal-related words group
apart from invertebrate/reptile words.

## Qualitative findings
Full detail in `qualitative_analysis.md`; the five themes:

1. **BERT's top scores are partly lexical overlap** — many of its best
   pairs (e.g. *toy poodle → poodle*, 0.891) contain the hypernym as a
   literal substring. A cleaner win is *gastropod → mollusk* (0.933).
2. **BERT fails on abstract taxonomic terms** — *sponge → invertebrate*
   (0.165), *bivalve → mollusk* (0.223), regardless of how obviously
   correct the pairing is.
3. **BERT fails on polysemy without context** — *dam → female* (0.162),
   because in isolation "dam" defaults to its water-barrier sense.
4. **GloVe's top scores are a vocabulary artifact** — all ten of its
   highest-scoring pairs score exactly 1.000 because an out-of-vocabulary
   modifier collapses the phrase onto its head noun (*apodiform bird* →
   *bird*). This is a vocabulary gap, not understanding.
5. **BERT genuinely beats GloVe on domain-specific role words** —
   *packhorse → pack animal* (0.766 vs −0.165), *mouser → domestic cat*
   (0.624 vs 0.004), where GloVe's single static vector is dominated by
   each word's most frequent everyday sense.

## Interpretation
Both embedding families carry a real but **partial and noisy** signal for
hypernymy: related pairs sit closer than unrelated ones, but no clean
geometric rule captures the relationship.

The central methodological finding is that **aggregate statistics alone
are misleading**. GloVe's larger effect size (d = 1.147 vs BERT's 0.565)
looks like a win for the simpler model until the qualitative analysis
shows it is substantially inflated by degenerate vocabulary-collapse
cases. BERT's smaller but cleaner effect reflects genuine semantic
association, alongside its own systematic failure modes on abstract terms
and polysemous words evaluated without context.

## Known limitations
- Words were embedded **out of context** (single words/phrases, not in a
  sentence), which is not how BERT is normally used and likely understates
  its capability.
- **GloVe phrase composition** is naive token averaging, which is the
  direct cause of the 1.000-similarity artifact.
- WordNet allows a word to have multiple parent categories (e.g. "dog" is
  reachable from "animal" via both a long biological path and a shorter
  "domestic animal" path); `depth` reflects the traversal path found, not
  a single true biological distance.
- PCA's 2D view captures only ~24% of the embedding space's variance —
  useful for intuition, not a full picture.
- Findings cover one sub-domain (animals) only.

## Possible next steps
- Embed words **in sentence context** rather than in isolation, which
  targets BERT's clearest failure mode (polysemy) directly.
- Train a classifier on the offset vectors to test whether hypernymy is
  linearly *separable* even without a single consistent direction.
- Extend the pipeline to a second sub-domain to test generalization.
- Use a subword-aware static model (e.g. fastText) to check whether
  GloVe's vocabulary-collapse artifact disappears.
