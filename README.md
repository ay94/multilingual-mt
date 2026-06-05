# multilingual-mt

Machine translation evaluation toolkit — benchmarking translation models across a range of languages, with automated metric evaluation (METEOR, BERTScore) and structured error analysis.

## Components

| File | Description |
|---|---|
| `template.ipynb` | Workflow template — dataset loading, translation, METEOR, BERTScore, error analysis. Adapt for any language pair. |
| `evaluation-template.md` | Structured template for documenting model selection, metric scores and error analysis findings per language |
| `WORKFLOW.md` | Full methodology — model types, computational benchmarks, metric explanations, language-specific considerations |
| `considerations.md` | Reference material — metric score examples with real translations, idiom examples, language script and parsing challenges, out-of-domain entity problem |
| `benchmarks/` | Per-language evaluation notebooks and notes |

## Evaluation workflow

Full methodology: [WORKFLOW.md](WORKFLOW.md)

Three-stage process:

1. **Exploratory** — identify a parallel benchmark corpus and candidate translation models for the target language
2. **Translation** — run candidate models, record outputs and computational performance (CPU vs GPU)
3. **Evaluation** — score with METEOR and BERTScore, sample low-scoring translations for manual error analysis

## Languages

Languages this workflow has been applied to:

- Afrikaans
- Arabic
- Bulgarian
- Bengali
- Czech
- Farsi
- French
- German
- Greek
- English
- Hausa
- Hindi
- Indonesian
- Japanese
- Malay
- Mandarin Chinese
- Portuguese
- Romanian
- Russian
- Serbo-Croatian
- Slovak
- Spanish
- Swahili
- Thai
- Turkish
- Twi
- Urdu
- Vietnamese
- Xhosa
- Zulu

## Benchmarks

| Language | Notebook | Notes |
|---|---|---|
| Afrikaans | [`benchmarks/af/`](benchmarks/af/) | mbart-large-50, madlad400, opus-mt-mul-en, opus-mt-gem-gem |
| Farsi | [`benchmarks/fa/`](benchmarks/fa/) | mbart-large-50 on MIZAN and PEPC |

See [`project-log.md`](project-log.md) for a running log of all evaluation runs across languages.

## Installation

```bash
pip install transformers sentencepiece bert_score sacrebleu nltk accelerate
```

## Quick start

```python
from transformers import MBartForConditionalGeneration, MBart50TokenizerFast

model     = MBartForConditionalGeneration.from_pretrained("facebook/mbart-large-50-many-to-many-mmt")
tokenizer = MBart50TokenizerFast.from_pretrained("facebook/mbart-large-50-many-to-many-mmt")
tokenizer.src_lang = "af_ZA"

inputs = tokenizer("Ek haat om te wag.", return_tensors="pt")
translated = model.generate(**inputs, forced_bos_token_id=tokenizer.lang_code_to_id["en_XX"])
print(tokenizer.decode(translated[0], skip_special_tokens=True))
# I hate waiting.
```

See [`template.ipynb`](template.ipynb) for the full evaluation workflow.
