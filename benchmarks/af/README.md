# MT Evaluation — Afrikaans

## Language

Afrikaans (`af`) · Latin script · High-resource relative to African languages

## Use case

Annotator-facing translation and unified-language topic modelling pre-processing.

## Benchmark dataset

**Tatoeba Afrikaans–English** (Helsinki-NLP)  
Public parallel corpus, not used in mbart-large-50 training split.  
Source: https://github.com/Helsinki-NLP/Tatoeba-Challenge

## Candidate models

| Model | HuggingFace ID | Type |
|---|---|---|
| mbart-large-50 | `facebook/mbart-large-50-many-to-many-mmt` | Many-to-many |
| madlad400-3b | `jbochi/madlad400-3b-mt` | Many-to-many |
| opus-mt-mul-en | `Helsinki-NLP/opus-mt-mul-en` | Many-to-one |
| opus-mt-gem-gem | `Helsinki-NLP/opus-mt-gem-gem` | Germanic family |

## Evaluation

### Benchmark notebook

[`afrikaans_mbart_benchmark.ipynb`](afrikaans_mbart_benchmark.ipynb) — mbart-large-50 on Tatoeba test data

### Observations

- **mbart-large-50** produces fluent, accurate translations for straightforward Afrikaans. Minor differences in wording (e.g. "I hate to wait" vs "I hate waiting") are within acceptable range.
- **madlad400-3b** showed inconsistency on simple sentences — some translations were correct, others were garbled or switched to a different language (French fragments appeared in Afrikaans output).
- **opus-mt-mul-en** and **opus-mt-gem-gem** performed well on sentence-level Tatoeba data; opus-mt-gem-gem requires `>>eng<<` prefix prepended to source text.
- All models struggle with idiomatic expressions and proper names not seen in training.

### Language-specific notes

- Afrikaans shares Germanic roots with Dutch and German — opus-mt-gem-gem (Germanic family model) is a viable option alongside the larger many-to-many models.
- No significant script or tokenisation challenges.
- Social media data introduces code switching with English, which affects all models.
