# Machine Translation Evaluation Workflow

This document outlines the evaluation workflow for machine translation in multilingual NLP projects. The workflow has three stages:

1. **Exploratory** — identify benchmark datasets and candidate translation models
2. **Translation** — run models, record outputs and computational performance
3. **Evaluation** — score translations using automated metrics, conduct error analysis

---

## Why machine translation is needed

Two distinct use cases drive MT in multilingual projects:

1. **Facilitating annotator analysis** — translating source language text into a language the analyst reads, so they can characterise topics and themes while the underlying analysis still runs on the original text.
2. **Unified language for topic modelling** — translating all source languages into a single target (typically English) so a single topic model can be run across the whole corpus. Useful when the source language lacks strong transformer support but has reliable translation models for English.

In both cases the goal is thematic analysis. The distinction is where translation sits: as an aid to the annotator, or as a pre-processing step for the backend pipeline.

---

## Model types

Translation models vary in scope:

| Type | Description |
|---|---|
| One-to-one | Translates one fixed language pair |
| One-to-many | One source language, target specified at inference |
| Many-to-one | Source specified at inference, one fixed target |
| Many-to-many | Both source and target specified at inference |
| Prompted generation | LLM prompted to translate |

More generic models (many-to-many) typically require larger architectures to maintain quality across language pairs. Speed largely depends on architecture and whether GPU is available.

**Practical speed benchmarks for `facebook/mbart-large-50-many-to-many-mmt`:**

| Setting | Batch size | Speed |
|---|---|---|
| CPU | 1 | 10–20 sec/instance |
| CPU | 16 | 4–6 sec/instance |
| CPU | 64 | 4–5 sec/instance |
| A100 GPU | 1 | 1–2 sec/instance |
| A100 GPU | 16 | 0.125–0.2 sec/instance |
| A100 GPU | 32 | 0.15–0.2 sec/instance (near memory limit) |

At CPU speeds, a dataset of 100,000 records takes 5–23 days. On A100 GPU, approximately 5 hours. GPU is required for any production-scale translation run.

Note: batching requires padding (`padding=True` when tokenising) to align sequence lengths within a batch.

---

## Stage 1 — Exploratory

1. **Benchmark dataset identification** — find a parallel corpus that represents the type of text in the project. It must not have been used in training the candidate models. The [Helsinki-NLP OPUS](https://github.com/Helsinki-NLP/OPUS-MT-train) and [Tatoeba Challenge](https://github.com/Helsinki-NLP/Tatoeba-Challenge) corpora are standard starting points.

   **Translation direction matters.** Most parallel corpora are created by translating from one language into another and then human-correcting the result. If you are evaluating Arabic→English translation and the benchmark was originally created in the English→Arabic direction (English source, Arabic translation), the "gold standard" English side is itself machine-generated and human-corrected — not a native English original. Evaluating against this introduces noise. Always check which direction a corpus was constructed in and ensure it matches the direction you are evaluating. This is a common source of misleadingly low scores — see the Turkish MT results as a documented example.

2. **Model exploration** — identify candidate models on HuggingFace for the target language pair. Consider:
   - Model type (one-to-one vs many-to-many)
   - Hosting options (local, HuggingFace Inference API, self-hosted TorchServe)
   - Language code format required by the model's tokeniser

---

## Stage 2 — Translation

1. Load the model and benchmark dataset
2. Run translations, recording outputs in the benchmark dataframe
3. Record computational performance: time per instance at different batch sizes, RAM/VRAM usage, CPU vs GPU

Translation output columns to record: `source`, `target` (gold standard), then one column per model.

---

## Stage 3 — Evaluation

### Automated metrics

**METEOR** (Metric for Evaluation of Translation with Explicit ORdering)
- Combines precision and recall with stemming, synonymy, and word order
- Handles paraphrases and variations in word order
- More sensitive to semantic similarity than BLEU
- Primary metric for projects where exact matches are not the goal

**BERTScore**
- Uses contextualised BERT embeddings to measure semantic similarity
- Greedy matching: for each token in the candidate, finds the best-matching token in the reference by cosine similarity
- Precision: average of best-match similarities for each candidate token
- Recall: average of best-match similarities for each reference token
- F1: harmonic mean of precision and recall
- Robust to paraphrasing and morphological variation — particularly useful for Arabic, Turkish, Urdu
- Can be applied monolingually (gold standard vs translation) or multilingually (source vs translation) depending on availability of gold standard
- **Alignment with topic modelling:** BERTScore operates in the same embedding space that topic models use to identify thematic similarity. This makes it particularly well-suited as the primary evaluation metric when translation is a pre-processing step for topic modelling — a translation that scores well on BERTScore is more likely to produce topic representations that are semantically consistent with the source, compared to a translation that scores well only on surface-level metrics like BLEU.

**mBERTScore** — multilingual variant of BERTScore, applied when comparing across languages (source vs translation) without a gold standard

**BLEU** — precision-oriented n-gram overlap. Suitable for projects requiring exact match evaluation. Systematically underestimates quality for morphologically rich languages.

**chrF** — character n-gram F-score. More sensitive than BLEU for morphological variation.

### Metric interpretation examples (METEOR)

| Range | What it indicates |
|---|---|
| > 0.7 | High semantic and structural accuracy |
| 0.6–0.7 | Minor variation in word choice or order |
| 0.5–0.6 | Partial meaning preserved, some paraphrase |
| < 0.5 | Significant meaning divergence |

### Metric interpretation examples (BERTScore F1)

| Range | What it indicates |
|---|---|
| > 0.95 | Near-identical semantic content |
| 0.90–0.95 | Strong semantic alignment, minor lexical variation |
| < 0.90 | Semantic divergence — review manually |

### Error analysis

1. Sample translations across metric score ranges — low, mid, high
2. For each sample: note what the model got right, what it missed, and why
3. Flag patterns: named entity preservation, tokenisation issues, idiomatic language, code switching, long sequence truncation
4. Compare scores across models to identify systematic differences

---

## Considerations

**Evaluation data quality** — check how the gold standard was produced. Literal translation corpora score differently from general-purpose ones. Low scores may reflect evaluation data quality, not model failure. Manual spot-checking of low-scoring examples is important.

**Training data overlap** — confirm the benchmark dataset was not used in model training. Many Helsinki-NLP models train on OPUS corpora — verify which splits were used.

**Out-of-domain data** — models trained on news articles may perform poorly on social media text. Domain shift affects all metrics. Named entities that did not appear in pre-training are particularly at risk — they may be transliterated incorrectly, dropped, or left untranslated.

**Sequence length** — models have a maximum token length. Long sentences may be truncated, causing loss of named entities or context at the end of the sequence. Monitor truncation as part of evaluation.

**Multilingual text (code switching)** — text mixing two languages (common in Indonesian, Tagalog, Arabic social media) is problematic for most models. Detect and flag these cases before evaluation.

**Language-specific challenges:**
- **Right-to-left scripts** (Arabic, Farsi, Urdu, Hebrew) — display and parsing differ; tokenisation boundaries may be affected
- **No word spacing** (Chinese, Japanese, Thai) — sentence splitting on punctuation does not work with Latin full stops
- **Contextual inference** (Korean, Chinese, Japanese) — subjects and other elements are often omitted when clear from context; sentence-level models may lose this context
- **Idiomatic language** — idioms rarely translate literally; a model unaware of an idiom will produce a literal translation with a completely different meaning and a misleading BERTScore
