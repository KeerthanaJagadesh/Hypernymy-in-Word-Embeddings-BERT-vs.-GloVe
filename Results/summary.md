# Hypernymy Project — Summary of Findings (Animal Domain)

## Goal
Test whether BERT's word embeddings capture hypernymy — the "is-a"
relationship between a specific word (hyponym, e.g. "poodle") and its
general category (hypernym, e.g. "dog") — using the animal domain as a
narrow, easy-to-verify test case.

## Method
1. **Dataset**: extracted hyponym-hypernym pairs from WordNet, starting at
   the `animal` synset and walking down 4 levels of its category tree.
   Result: **514 pairs**, **507 unique words/phrases**, spanning depths 0-3
   (depth = how many WordNet steps below "animal" the pair sits).
2. **Embeddings**: fed each unique word through pretrained `bert-base-uncased`,
   mean-pooling its subword token vectors into a single 768-dimensional
   vector per word.
3. **Analysis**:
   - Cosine similarity between each true pair's vectors, compared against
     a random-pair control group (same words, mismatched combinations).
   - "Offset consistency": whether `general_vector - specific_vector` points
     in a similar direction across all pairs (the same trick behind
     word2vec analogies like king − man + woman ≈ queen).
   - 2D PCA visualization of all 507 word vectors, colored by depth.

## Results

| Metric | True pairs | Random pairs |
|---|---|---|
| Mean cosine similarity | **0.508** | 0.426 |
| Std dev | 0.166 | 0.122 |

- True hypernym/hyponym pairs are **measurably more similar** than random
  word pairs, showing BERT does encode some relatedness signal for these
  category relationships.
- **Offset consistency: 0.209** (out of 1.0) — no single consistent
  "hypernymy direction" exists across pairs. The relationship is present
  but not linearly encoded the way some other word relationships are.
- Similarity by depth was fairly flat (~0.46–0.52 across depths 0-3, see
  `depth_similarity.png`), with heavily overlapping error bars — meaning
  there is no statistically meaningful trend showing the model does
  better on close vs. distant category relationships.
- PCA visualization (2D, capturing only 23.7% of total variance) showed
  loose but visible semantic clustering — e.g. mammal-related words
  grouped together, separate from invertebrate/reptile words.

## Interpretation
BERT embeddings carry a real but noisy signal for hypernymy: related
pairs are on average closer together than unrelated pairs, but there is
no clean geometric rule (like a single consistent vector offset) that
captures the relationship. This lines up with broader NLP findings that
contextual embeddings like BERT encode relational structure in a
distributed, non-linear way rather than as simple vector arithmetic.

## Known limitations
- Words were embedded **out of context** (single words/phrases fed to
  BERT alone, not in a sentence), which is not how BERT is normally used
  and may weaken the quality of its representations.
- WordNet allows a word to have multiple parent categories (e.g. "dog"
  is reachable from "animal" via both a long biological path and a
  shorter "domestic animal" path); `depth` reflects the traversal path
  found, not a single true biological distance.
- PCA's 2D view captures only ~24% of the embedding space's variance —
  useful for intuition, not a full picture.

## Possible next steps
- Repeat the analysis with a second embedding model (e.g. a
  sentence-transformer) to see if the "no consistent direction" finding
  holds generally or is specific to BERT.
- Train a simple classifier on the offset vectors to see if hypernymy is
  linearly *separable* even if not a single consistent direction.
- Extend the same pipeline to a second sub-domain for comparison.
