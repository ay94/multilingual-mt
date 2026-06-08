# MT Evaluation — Turkish

## Language

Turkish (`tr`) · Latin script · Agglutinative morphology

## Use case

- [x] Unified-language translation (pre-processing for topic modelling)
- [x] Annotator-facing translation (facilitating analysis)

## Benchmark dataset

**MaCoCu Turkish–English** (`MaCoCu-tr-en.sent.txt`)  
Parallel corpus of 1.6M sentence pairs crawled from Turkish-language web domains in 2021.  
Source: https://macocu.eu/

Filtering applied before evaluation (retains highest-quality pairs):

| Filter | Condition | Retained |
|---|---|---|
| Translation direction | `first-orig-second-ht` (Turkish original, English human translation) | ~808K pairs |
| `bicleaner_ai_score` | > 0.9 | Removes low-quality translation candidates |
| `bleualign_score` | > 0.4 | Removes poorly aligned sentence pairs |
| Sample | Random 10,000 | Final evaluation set |

Note: the low-BERTScore error sample contained Chinese-language source sentences — MaCoCu includes multilingual content and the language filters are not perfect. This is a data quality issue, not a model failure. Verify source language before drawing error analysis conclusions.

## Candidate models

| Model | HuggingFace ID | Type |
|---|---|---|
| MBART many-to-many | `facebook/mbart-large-50-many-to-many-mmt` | Many-to-many |
| Helsinki opus-mt-tr-en | `Helsinki-NLP/opus-mt-tr-en` | One-to-one |

---

## Translation stage

### Model configuration

| Model | Source lang code | Target lang code | Max length | Batch size |
|---|---|---|---|---|
| MBART | `tr_TR` | `en_XX` | 512 | 16 |
| Helsinki | `tr_TR` (Marian) | `en` | 512 | 16 |

### Computational performance (GPU A100)

| Model | Batch size | GPU time (10K samples) | CPU estimate (10K samples) |
|---|---|---|---|
| MBART | 16 | 41 min | ~23 hrs |
| Helsinki | 16 | 20 min | ~4 hrs |

CPU iteration rates observed: MBART ~135s/batch, Helsinki ~25s/batch.

---

## Evaluation

Three metrics are used:

| Metric | What it compares | Sensitivity |
|---|---|---|
| **METEOR** | Translation vs gold standard (lexical + stemming + synonymy) | High — penalises paraphrasing |
| **BERTScore** | Translation vs gold standard (semantic, English embeddings) | Moderate |
| **XLMrScore** | Translation vs source text (cross-lingual, `xlm-roberta-base`) | Cross-lingual meaning preservation |

### Scores

| Model | METEOR | BERTScore F1 | XLMrScore F1 |
|---|---|---|---|
| MBART many-to-many | **0.534** | **0.929** | **0.852** |
| Helsinki opus-mt-tr-en | 0.500 | 0.929 | 0.852 |

Descriptive statistics (MBART):

| Stat | METEOR | BERTScore | XLMrScore |
|---|---|---|---|
| Mean | 0.534 | 0.929 | 0.852 |
| Std | 0.178 | 0.025 | 0.026 |
| Min | 0.000 | 0.790 | 0.745 |
| 25% | 0.427 | 0.915 | 0.836 |
| Median | 0.531 | 0.928 | 0.849 |
| 75% | 0.639 | 0.942 | 0.866 |

### Score interpretation

- BERTScore and XLMrScore are nearly identical across both models. The gap between MBART and Helsinki is visible only in METEOR (0.534 vs 0.500 — a difference of 0.034).
- The small std on BERTScore and XLMrScore indicates consistent semantic preservation across the corpus — models do not degrade dramatically on outliers.
- High BERTScore / moderate METEOR is the dominant pattern: translations preserve meaning but rephrase structure. This is expected and acceptable for topic modelling — thematic content is preserved even when surface form differs.
- Low METEOR with high BERTScore cases warrant manual review — occasionally BERTScore stays high even on incorrect translations because token-level matching does not catch complete semantic failures.

### Error analysis samples

**High scoring (BERTScore > 0.96, METEOR > 0.85)**

| Turkish | Gold standard | MBART |
|---|---|---|
| Ülkemiz, Bursa ve Gürsu çapında… Eurodesk Temas Noktası oluşturuldu | With the projects… the Eurodesk Contact Point was established by Gürsu Municipality in 2019 | Through the projects… a Eurodesk Contact Point was established in 2019 by the Municipality of Gürsu |
| ÖSYM tarafından yapılan… en az 55 standart puan almış olmak | Having taken at least 55 standard points… of the Academic Staff and Graduate Education Entrance Exam | to have received at least 55 standard marks… of the Academic Staff and Postgraduate Training Initial Examination |

**Mid scoring (BERTScore ~0.93, METEOR ~0.49)**

| Turkish | Gold standard | MBART | Notes |
|---|---|---|---|
| Isı yalıtımı ve dış cephe mantolama… | Thermal insulation and exterior sheathing… | Irrigation and outdoor shelter… | Domain terminology failure — "ısı yalıtımı" (thermal insulation) translated as "irrigation" |
| IETM toplantıları daha önce görmediğiniz… | IETM meetings are full of inspiration… | IETM meetings are an excellent opportunity to go to a country you have never seen… | Paraphrase — correct meaning, different structure |

**Low scoring (BERTScore < 0.90, METEOR < 0.2)**

| Turkish | Gold standard | MBART | Notes |
|---|---|---|---|
| Chinese characters (data quality) | Correct English | Garbled output | MaCoCu data quality issue — Chinese in Turkish corpus |

### Error patterns observed

- **Named entity preservation:** proper names generally preserved; inconsistent on transliteration (Layla/Lily, Sami/Sammy)
- **Idiomatic language:** moderate failures — idioms translated literally with semantically incorrect result
- **Terminology:** domain-specific vocabulary is a known weakness (thermal insulation example above)
- **Data quality:** low-BERTScore sample contained non-Turkish source text (Chinese) — filter language detection more strictly if MaCoCu is reused
- **Helsinki-specific:** Helsinki showed slightly lower METEOR across all examples reviewed; BERTScore gap is negligible

---

## Recommendation

**MBART is recommended** for Turkish→English translation. The METEOR gap (0.034) is meaningful given METEOR's sensitivity — a 0.03 improvement at the mean corresponds to noticeably more faithful sentence-level translations.

If compute is the primary constraint (e.g. large volume on CPU, no GPU), Helsinki is an acceptable fallback — the semantic content captured by BERTScore and XLMrScore is equivalent, and topic modelling quality is unlikely to be affected significantly.

---

## Notebooks

| Notebook | Description |
|---|---|
| [`turkish_mbart_benchmark.ipynb`](turkish_mbart_benchmark.ipynb) | MBART many-to-many on MaCoCu |
| [`turkish_helsinki_benchmark.ipynb`](turkish_helsinki_benchmark.ipynb) | Helsinki opus-mt-tr-en on same data |
