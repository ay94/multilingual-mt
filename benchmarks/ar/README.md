# MT Evaluation — Arabic

## Language

Arabic (`ar`) · Arabic script · Right-to-left · Morphologically rich

## Use case

- [x] Unified-language translation (pre-processing for topic modelling)
- [x] Annotator-facing translation (facilitating analysis)

## Benchmark datasets

Two datasets evaluated at different stages.

**OPUS test set** (`opus-2019-12-18.test.txt`)  
5,000 Arabic–English sentence pairs. General-domain, sentence-level parallel text.  
Source: https://opus.nlpl.eu/  
Used for: initial model selection.

**UN Parallel Corpus** (`un_pc`, Arabic–English split)  
10,000 sentence pairs from 25 years of UN documents (1990–2014), loaded via HuggingFace `datasets`.  
Source: https://huggingface.co/datasets/un_pc  
Used for: topic modelling evaluation (domain closer to social analysis datasets).

Note on translation direction: most OPUS and UN corpus pairs were produced by translating from English into Arabic then human-correcting — the "gold standard" English side is not a native English original. This introduces noise at lower METEOR scores and should be accounted for when interpreting results. Spot-check low-scoring examples before concluding a model has failed.

## Candidate models

| Model | HuggingFace ID | Type |
|---|---|---|
| MBART many-to-many | `facebook/mbart-large-50-many-to-many-mmt` | Many-to-many |

Other models were tested during exploratory evaluation; most produced noisy output with special characters and were excluded. MBART was the only model producing consistently usable translations.

---

## Translation stage

### Model configuration

| Model | Source lang code | Target lang code | Max length | Batch size |
|---|---|---|---|---|
| MBART | `ar_AR` | `en_XX` | 512 | 4–8 |
| MBART | `ar_AR` | `en_XX` | 1024 | 8 |

### Computational performance

GPU required for any production-scale run. CPU throughput on Arabic with MBART at max_length=1024 is not practical for datasets > 5,000 sentences.

---

## Evaluation

### Scores

#### OPUS test set (5,000 sentence pairs)

| Model | METEOR | BERTScore F1 |
|---|---|---|
| MBART many-to-many | **0.678** | **0.964** |

Descriptive statistics:

| Stat | METEOR | BERTScore |
|---|---|---|
| Mean | 0.678 | 0.964 |
| Std | 0.302 | 0.034 |
| Min | 0.000 | 0.792 |
| 25% | 0.483 | 0.941 |
| Median | 0.736 | 0.968 |
| 75% | 0.981 | 1.000 |

#### UN parallel corpus (10,000 sentence pairs)

| Model | METEOR | BERTScore F1 | XLMrScore (ar→tr) | XLMrScore (en→tr) |
|---|---|---|---|---|
| MBART many-to-many | **0.405** | **0.916** | 0.834 | 0.899 |

Lower METEOR on the UN corpus reflects the domain shift to formal policy and administrative language — specialised terminology is a known weakness.

### Score interpretation

- The high BERTScore (0.964 on OPUS) indicates strong semantic preservation — the core meaning of Arabic sentences is reliably transferred to English.
- The wide METEOR spread (std=0.302) reflects METEOR's sensitivity: sentences with paraphrasing, pronoun substitution, or structural changes score much lower even when meaning is correct.
- The METEOR/BERTScore divergence is intentional and expected: topic modelling depends on semantic content, not surface form. BERTScore is the more relevant metric for this use case.
- A combined average of METEOR and BERTScore provides a more balanced signal when the two diverge significantly.

### Error analysis

Key patterns from manual inspection of 30 low-scoring sentences (METEOR < 0.5 or BERTScore < 0.8):

**Paraphrase / structural changes** (METEOR impact, BERTScore unaffected)

| Arabic | Gold standard | MBART |
|---|---|---|
| حُكِمَ على فاضل بالإعدام لقتله لفتاة صغيرة | Fadil was sentenced to death for killing a little girl | He was executed for murdering a little girl |
| لا يستطيعُ توم شراء ما يريد… | Tom can't buy what he wants… | Tom can't buy what he wants… |

MBART rephrases actively: "was sentenced to death for killing" → "was executed for murdering" — same meaning, different words. METEOR penalises this; BERTScore does not.

**Named entity handling**

| Arabic | Gold standard | MBART | Note |
|---|---|---|---|
| ليلى | Layla | Lily | Transliteration inconsistency |
| فاضل | Fadil | Virtue | Semantic translation of name rather than transliteration |

Arabic names can be transliterated or semantically translated — both are technically defensible but create mismatches with gold standard transliterations.

**High BERTScore, low METEOR — semantic but not lexical**

| Arabic | Gold standard | MBART | METEOR | BERTScore |
|---|---|---|---|---|
| انتظر يا جمال | Wait, Jamal | Hold on, Beauty | 0.000 | 0.912 |
| هذا المكان رائع | This place is great | It's a wonderful place | 0.000 | 0.891 |

These cases show BERTScore remaining high while METEOR is near zero — the models correctly preserve meaning but use completely different vocabulary.

### Error patterns observed

- **Named entity preservation:** inconsistent transliteration vs semantic translation of Arabic names; proper nouns from specialised domains (UN agencies, legal terms) sometimes left in Arabic or incorrectly translated
- **Idiomatic language:** Arabic idioms often produce literal translations with different meaning
- **Right-to-left script:** no display issues detected in evaluation pipeline; tokenisation boundaries are handled correctly by MBART's Arabic-trained BPE
- **Pronoun dropping:** Arabic often drops subject pronouns — MBART infers correctly in most cases
- **Gold standard quality (UN corpus):** UN parallel corpus was created English-first → translated to Arabic → back-translated; gold standard English is therefore an indirect translation, introducing noise especially at lower METEOR ranges

---

## Recommendation

**MBART is the recommended model for Arabic→English translation.** No other tested model produced usable output consistently. MBART achieves strong BERTScore (0.964 on OPUS), which is the primary relevant metric for topic modelling applications.

METEOR=0.678 on OPUS is acceptable for the use case — the residual gap from perfect alignment is attributable to paraphrasing, not semantic failure.

For UN-style formal documents, expect METEOR ~0.40 — the model handles general Arabic well but specialised terminology is a known gap.

---

## Notebooks

| Notebook | Description |
|---|---|
| [`arabic_mbart_benchmark.ipynb`](arabic_mbart_benchmark.ipynb) | MBART many-to-many on OPUS test set |
