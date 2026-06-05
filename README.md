# multilingual-mt

Machine translation evaluation toolkit — benchmarking translation models across a range of languages, with automated metric evaluation (METEOR, BERTScore, mBERTScore) and structured error analysis.

## Documentation

| File | Description |
|---|---|
| [`WORKFLOW.md`](WORKFLOW.md) | End-to-end methodology — model types, deployment options, computational benchmarks, three-stage workflow, metric selection by use case |
| [`metrics.md`](metrics.md) | Metrics reference — BLEU, METEOR, TER, ROUGE, BERTScore: how each works, when to use it, with references |
| [`considerations.md`](considerations.md) | Reference material — metric score examples with real translations, evaluation data caveats, idiom challenges, language script and parsing issues, out-of-domain entity problem |
| [`evaluation-template.md`](evaluation-template.md) | Structured template for documenting model selection, computational results, metric scores and error analysis per language |

## Notebooks

| File | Description |
|---|---|
| [`template.ipynb`](template.ipynb) | Workflow template — dataset loading, translation, METEOR, BERTScore, mBERTScore, error analysis. Adapt for any language pair. |

## Benchmarks

| Language | Notes |
|---|---|
| [Afrikaans](benchmarks/af/) | mbart-large-50 on Tatoeba |
| [Farsi](benchmarks/fa/) | mbart-large-50 on MIZAN and PEPC |
| [Spanish](benchmarks/es/) | mBART vs Helsinki on Europarl + OPUS — Helsinki recommended |

## Languages

Languages this workflow has been applied to:

- Afrikaans
- Arabic
- Bengali
- Bulgarian
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

## Installation

```bash
pip install transformers sentencepiece bert_score sacrebleu nltk accelerate
```

## Quick start

```python
from transformers import MBartForConditionalGeneration, MBart50TokenizerFast

model     = MBartForConditionalGeneration.from_pretrained("facebook/mbart-large-50-many-to-many-mmt")
tokenizer = MBart50TokenizerFast.from_pretrained("facebook/mbart-large-50-many-to-many-mmt")
tokenizer.src_lang = "es_XX"

inputs     = tokenizer("Hola, ¿cómo estás?", return_tensors="pt")
translated = model.generate(**inputs, forced_bos_token_id=tokenizer.lang_code_to_id["en_XX"])
print(tokenizer.decode(translated[0], skip_special_tokens=True))
# Hello, how are you?
```

See [`template.ipynb`](template.ipynb) for the full evaluation workflow.
