# MT Evaluation — Spanish

## Language

Spanish (`es`) · Latin script

## Benchmark datasets

Two public parallel corpora evaluated:

**Europarl Corpus** — proceedings of the European Parliament, 21 European languages.  
Source: https://www.statmt.org/europarl/  
Chosen for its complexity, diversity of topics, and specialised terminology — more representative of the types of content likely encountered in project data than simpler corpora.

**OPUS MT Test** — the evaluation benchmark used by Helsinki-NLP to evaluate their models.  
Source: https://huggingface.co/Helsinki-NLP/opus-mt-es-en  
Contains shorter, more direct sentences. Used as a secondary reference.

**Dataset choice:** Europarl was selected as the primary benchmark. Its complexity and topic diversity make it a more challenging and realistic evaluation, more aligned with the analytical use case. Both datasets showed good source–target alignment.

## Models evaluated

| Model | HuggingFace ID | Type |
|---|---|---|
| mBART | `facebook/mbart-large-50-many-to-many-mmt` | Many-to-many |
| Helsinki | `Helsinki-NLP/opus-mt-es-en` | One-to-one (es→en) |

## Computational performance (Tesla V100-SXM2-16GB, batch size 16)

| Model | Dataset | Samples | Time | Speed | Memory |
|---|---|---|---|---|---|
| mBART | Europarl | 1000 | 185.92s | 2.95 it/s | 14,926 MiB |
| mBART | OPUS | 1000 | 75.82s | 1.20 it/s | 14,926 MiB |
| Helsinki | Europarl | 1000 | 68.9s | 1.09 it/s | 13,066 MiB |
| Helsinki | OPUS | 1000 | 23.97s | 0.38 it/s | 9,394 MiB |

**Production run (Helsinki, project data):** 21,409 samples translated in 4,931 seconds (82 min) at 3.68 it/s. Memory: 13,066 MiB.

Helsinki is substantially faster and uses less memory than mBART across all conditions.

## Model comparison

### Named entity handling
Both models handle named entities effectively. Helsinki is particularly strong in maintaining accuracy of proper nouns and organisation names.

### Gist vs literal translation

**mBART** — stays close to the source sentence structure and word choice. More suitable when literal translation is required, but occasionally introduces content not in the original (hallucination). See instance 75 in europarl evaluation samples as a documented example.

**Helsinki** — maintains core meaning with subtle shifts in wording that do not alter meaning. Balances gist and literal translation, leaning towards overall meaning conversion. More flexible without sacrificing accuracy.

### mBART known limitation
mBART shows unreliable behaviour on Spanish→English specifically — this is a documented community-reported issue with the model, not isolated to this evaluation. Use with caution for Spanish→English; Helsinki is more reliable for this direction.

## Recommendation

**Helsinki (`opus-mt-es-en`)** is recommended for Spanish→English translation. It is more fluid, faster, lower memory, and more reliable than mBART for this language pair. mBART is capable but requires validation due to occasional errors and the known Spanish→English reliability issue.

## Evaluation notebooks

| Notebook | Model |
|---|---|
| [`spanish_mbart_benchmark.ipynb`](spanish_mbart_benchmark.ipynb) | facebook/mbart-large-50-many-to-many-mmt |
| [`spanish_helsinki_benchmark.ipynb`](spanish_helsinki_benchmark.ipynb) | Helsinki-NLP/opus-mt-es-en |
