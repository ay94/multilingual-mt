# MT Evaluation Metrics

Reference document covering the automated metrics used in machine translation evaluation — what each metric measures, how it works, and when it is most appropriate.

---

## Choosing a metric

| Project objective | Recommended metrics |
|---|---|
| Exact match / precision-focused | BLEU, ROUGE |
| Broader semantic alignment / general content | METEOR, BERTScore |
| Both precision and semantic alignment | BERTScore (dual capability — works against gold standard or source text) |

BERTScore is the most versatile: it can evaluate semantic similarity between translation and gold standard (monolingual) and between translation and source text (multilingual, when no gold standard is available).

---

## BLEU (BiLingual Evaluation Understudy)

Primarily focuses on precision — the fraction of words in the machine translation that also appear in the reference translation. Uses n-gram precision to compare overlap between the translation and the reference.

- Does not account for synonyms or semantic similarity — strictly checks exact word matches
- Typically operates at corpus level, evaluating a set of sentences together rather than individually
- Uses n-gram matching with varying n (unigrams, bigrams, trigrams, etc.)
- Systematically underestimates quality for morphologically rich languages (Arabic, Turkish, Urdu) where the same meaning has many valid surface forms

**Use when:** exact match is the priority and the language is morphologically simple.

---

## METEOR (Metric for Evaluation of Translation with Explicit ORdering)

Combines precision and recall with linguistic factors: stemming, synonymy, and word order alignment.

- Considers both exact and partial matches — allows flexibility in word choice
- Captures semantic similarity through synonym matching and stemming
- Handles paraphrases and variations in word order better than BLEU
- Evaluates at sentence level, unlike BLEU which operates at corpus level

**Use when:** the goal is to capture the general content and context of source text rather than exact matches. Primary metric for topic modelling pre-processing use cases.

---

## TER (Translation Edit Rate)

Measures the number of edits (insertions, deletions, substitutions) required to transform the machine translation output into the reference translation.

- Focuses on edit distance between translation and reference
- Provides a measure of fluency and adequacy
- Sensitive to minor differences in word order and word choice
- Useful for sentence-level evaluation

---

## ROUGE (Recall-Oriented Understudy for Gisting Evaluation)

A set of metrics measuring n-gram overlap between the machine-generated translation and the reference. Commonly used in text summarisation and machine translation.

Variants:
- **ROUGE-N** — n-gram overlap
- **ROUGE-L** — longest common subsequence
- **ROUGE-S** — skip-bigram overlap

**Use when:** recall matters as much as precision, or for summarisation tasks alongside translation.

---

## BERTScore

Designed to evaluate translation quality using pre-trained BERT embeddings, capturing semantic similarity between candidate and reference texts beyond surface-level overlap.

### Mechanism

Uses BERT embeddings to map both candidate and reference texts into high-dimensional vectors. Calculates cosine similarity between token representations using a greedy matching strategy.

### Components

- **Precision** — for each token in the candidate, find the best-matching token in the reference by cosine similarity; average across all candidate tokens
- **Recall** — for each token in the reference, find the best-matching token in the candidate; average across all reference tokens
- **F1** — harmonic mean of precision and recall

### How greedy matching works

For each token in the candidate sentence, BERTScore finds the token in the reference with the highest cosine similarity. This is computed in both directions (candidate → reference for precision, reference → candidate for recall), then combined into F1.

### Advantages

- Goes beyond lexical overlap — captures semantic meaning
- Sensitive to paraphrasing — appropriate when different wordings convey the same meaning
- Applicable both monolingually (translation vs gold standard) and multilingually (translation vs source text)
- Aligned with embedding-based downstream tasks: a translation that scores well on BERTScore is more likely to produce consistent representations in topic modelling and NER, which also operate in embedding space

---

## References

- Zhang et al. (2020) — [BERTScore: Evaluating Text Generation with BERT](https://arxiv.org/abs/1904.09675)
- [BERTScore language list and supported models](https://github.com/Tiiiger/bert_score)
- [Monitoring Text-Based Generative AI Models Using Metrics Like BLEU Score](https://arize.com/blog-course/generative-ai-metrics-bleu-score/)
- [How to Evaluate Text Generation Models — Metrics for Automatic Evaluation of NLP Models](https://towardsdatascience.com/how-to-evaluate-text-generation-models-metrics-for-automatic-evaluation-of-nlp-models-e1c251b04ec1)
- [METEOR metric for machine translation](https://machinelearninginterview.com/topics/machine-learning/meteor-for-machine-translation/)
- Banerjee & Lavie (2005) — [METEOR: An Automatic Metric for MT Evaluation with Improved Correlation with Human Judgments](https://aclanthology.org/W05-0909.pdf)
