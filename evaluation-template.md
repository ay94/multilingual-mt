# MT Evaluation Template — [Language]

Use this template to document the evaluation for each target language. Fill in each section as you progress through the three-stage workflow.

---

## Language

[Language name, ISO code, and script]

## Use case

- [ ] Annotator-facing translation (facilitating analysis)
- [ ] Unified-language translation (pre-processing for topic modelling / NER)

## Benchmark dataset

[Dataset name, source, size, and link. Note whether it was used in training any candidate models.]

## Candidate models

| Model | HuggingFace ID | Type | License |
|---|---|---|---|
| | | | |

---

## Translation stage

### Model configuration

| Model | Source lang code | Target lang code | Batch size | Notes |
|---|---|---|---|---|
| | | | | |

### Computational performance

| Model | Setting | Batch size | Speed (sec/instance) | RAM/VRAM |
|---|---|---|---|---|
| | CPU | | | |
| | GPU | | | |

---

## Evaluation

### Automated metric scores

| Model | METEOR | BERTScore F1 | mBERTScore | Notes |
|---|---|---|---|---|
| | | | | |

### Score interpretation

[Which model performs best? Any divergences between METEOR and BERTScore? What do the scores indicate for this language pair and domain?]

### Error analysis samples

Draw samples across score ranges and document patterns:

**High scoring (BERTScore > 0.95 / METEOR > 0.7)**

| Source | Gold standard | Translation | Notes |
|---|---|---|---|
| | | | |

**Mid scoring (BERTScore 0.90–0.95 / METEOR 0.5–0.7)**

| Source | Gold standard | Translation | Notes |
|---|---|---|---|
| | | | |

**Low scoring (BERTScore < 0.90 / METEOR < 0.5)**

| Source | Gold standard | Translation | Notes |
|---|---|---|---|
| | | | |

### Error patterns observed

- Named entity preservation:
- Idiomatic language:
- Code switching:
- Tokenisation / truncation:
- Domain shift (benchmark vs project data):

---

## Recommendation

[Which model is recommended for this project and why? What caveats apply?]

---

## Considerations / observations

[Any language-specific challenges, post-processing requirements, or notes for future evaluations.]
