# Qualitative Analysis: BERT vs. GloVe on Animal Hypernym Pairs

## 1. BERT's top scores are partly driven by lexical overlap, not semantics

| specific | general | BERT sim |
|---|---|---|
| forest tent caterpillar | tent caterpillar | 0.953 |
| edible sea urchin | sea urchin | 0.935 |
| pink bollworm | bollworm | 0.912 |
| miniature poodle | poodle | 0.891 |
| toy poodle | poodle | 0.884 |

Most of BERT's highest-confidence pairs are cases where the hypernym word
appears *literally as a substring* of the hyponym phrase (e.g. "toy poodle"
contains "poodle"). This inflates apparent success: the model may be
detecting textual overlap rather than a genuine hierarchical "is-a"
relationship. A cleaner success case without lexical overlap is
**gastropod → mollusk (0.933)** — a real semantic win.

## 2. BERT's clearest failure mode: abstract taxonomic category words

| specific | general | BERT sim |
|---|---|---|
| sponge | invertebrate | 0.165 |
| worm | invertebrate | 0.211 |
| oyster | bivalve | 0.222 |
| bivalve | mollusk | 0.223 |

BERT consistently struggles when the hypernym itself is a rare, technical
biological classification term (invertebrate, bivalve, mollusk) rather than
an everyday word like "dog" or "bird" — regardless of how obviously correct
the pairing is to a human reader with domain knowledge.

## 3. BERT's second failure mode: polysemous everyday words

| specific | general | BERT sim | likely cause |
|---|---|---|---|
| dam | female | 0.162 | "dam" defaults to "water barrier," not "female animal parent" |
| finisher | racer | 0.162 | "finisher" defaults to general/sports sense, not horse racing |
| grub | larva | 0.215 | "grub" defaults to slang for "food" |

Since words were embedded in isolation (no sentence context), BERT falls
back on each word's dominant everyday meaning. When the WordNet sense is a
rare, domain-specific meaning, the embedding reflects the wrong sense
entirely — a direct consequence of the out-of-context embedding limitation
already noted in the main summary.

## 4. GloVe's top scores are almost entirely a vocabulary-coverage artifact

| specific | general | GloVe sim |
|---|---|---|
| apodiform bird | bird | 1.000 |
| cuculiform bird | bird | 1.000 |
| branchiopod crustacean | crustacean | 1.000 |
| liver-spotted dalmatian | dalmatian | 1.000 |

This is the single most important qualitative caveat of the study. GloVe has
no vector for rare modifiers like "apodiform" or "cuculiform," so averaging
collapses the whole phrase onto the head noun's vector exactly, producing a
perfect similarity of 1.0. **This is not the model understanding the
relationship — it is a vocabulary gap disguised as a perfect score.** All
ten of GloVe's top-10 pairs are affected by this artifact (see
`results/model_comparison.csv`, `n_degenerate_pairs`).

## 5. Where BERT clearly beats GloVe: domain-specific animal-role words

| specific | general | BERT sim | GloVe sim |
|---|---|---|---|
| packhorse | pack animal | 0.766 | −0.165 |
| maverick | calf | 0.782 | 0.034 |
| mouser | domestic cat | 0.624 | 0.004 |
| mastiff | working dog | 0.726 | 0.078 |
| worker | insect | 0.663 | 0.065 |

These are words with a specialized animal-related sense competing against a
much more common unrelated sense ("maverick" = independent-minded person;
"worker" = employee). BERT still recovers meaningful similarity here, while
GloVe's single static vector per word is dominated by the word's
globally most frequent usage — a genuine, non-artifactual advantage for
BERT's subword/contextual architecture on this kind of vocabulary.

## Takeaway

Once the GloVe vocabulary-collapse artifact is accounted for, the picture is
more nuanced than the raw similarity numbers suggest: GloVe's headline
statistical advantage (Section on Quantitative Results) is partly inflated
by degenerate cases, while BERT shows genuine, interpretable advantages on
domain-specific vocabulary — but also has clear, systematic failure modes on
abstract taxonomic terms and polysemous words evaluated without context.
