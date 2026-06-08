# MT Evaluation — Considerations & Language Challenges

Reference material for machine translation evaluation. Covers metric interpretation with concrete examples, evaluation data caveats, out-of-domain data, and language-specific challenges.

---

## Overview

When evaluating machine translation models for NLP tasks, translation accuracy can be broken down into several sub-categories, each requiring different objective functions to measure:

- **Grammar / Fluency** — the natural flow of the translated text; how realistic and human the translation feels
- **Semantic accuracy** — preservation of meaning; the translation maintains the same semantics as the original
- **Idiomatic language** — the capacity to translate idiomatic concepts that may not have a direct translation into English
- **Named entity translation** — the ability to translate entity names correctly; mostly relevant to languages using non-Latin scripts, where names may be phonetic translations

Each of these can have a significant impact on downstream NLP task performance. For topic modelling specifically, it is important that defining characteristics within the text — common entity names, and words relevant to the various themes — are translated accurately so that they cluster together correctly. A translation error on a key entity or theme word can scatter mentions across topics rather than grouping them.

---

## Metric score interpretation

### METEOR examples

| Range | Source (Chinese) | Gold standard | Translation |
|---|---|---|---|
| > 0.7 | 我希望我能够看我的孩子长大 | I only wish I could see my kids grow up. | I wish I could see my children grow up |
| 0.6–0.7 | 然后我思考了很多关于死亡的事 | And I thought about death a lot. | Then I thought a lot about death |
| 0.5–0.6 | 来，让我给你打个比方。 | Here, let me give you an analogy. | Let me give you a comparison. |
| < 0.5 | 回来时已经濒临死亡。 | She was back and near death. | When he came back, he was on the verge of death. |

Translations with near-identical structure and synonymous words achieve around 0.7 in the best case. The score drops quickly as structural changes increase — yet in all four examples above, the semantics are still accurately preserved. The last example illustrates how subject-dropping languages (Chinese omits the subject when inferable from context) can cause a mismatched pronoun in the translation ("she" → "he"), pushing the score below 0.5 despite the overall meaning being correct.

### BERTScore examples

| Range | Source (Chinese) | Gold standard | Translation |
|---|---|---|---|
| 0.95–1.0 | 这一次，一个邻居看到了。 | And in this case, a neighbor saw it. | This time, a neighbor saw it. |
| 0.90–0.95 | 因为正值夏天，我穿的是短裤 | It was summertime: I had shorts on. | Because it's summertime, I wear shorts |
| < 0.90 | 我们会努力追赶 | We could align ourselves with it. | We'll try to catch up |

The first example is semantically close but uses different words and a reduced structure — BERTScore handles this well. The other two examples highlight a known limitation: despite being slightly off in matching the semantics of the gold standard, they maintain relatively high scores. This suggests a threshold somewhere in the 0.90–0.95 range where semantic accuracy begins to taper off. **Note:** the last example's gold standard ("We could align ourselves with it") is not actually an accurate translation of the source — a good reminder that low scores can reflect gold standard quality rather than model failure, and manual review is necessary before drawing conclusions.

The lowest observed BERTScore in this dataset was approximately 0.83. Scores below 0.90 warrant manual review.

---

## Evaluation data caveats

### Evaluation data quality

Benchmark corpora vary in how they were produced. Some were created with the goal of providing literal, word-for-word translations; others provide more natural, idiomatic translations that deviate in surface form while preserving meaning. This directly affects metric scores — a good translation compared against a loose gold standard will score lower than it deserves.

**Verification principle:** never draw conclusions from low scores alone. Always manually inspect a sample of low-scoring translations to determine whether the model is making mistakes or whether the gold standard is the issue. This is particularly important for low-resource languages where benchmark datasets may be small or inconsistently curated.

### Training data overlap

Many translation models (especially one-to-one Helsinki-NLP models) are trained on OPUS corpora. When selecting a benchmark, verify that the model was not trained on it — or at minimum, use only the held-out test split. The OPUS series provides matching evaluation corpora for this purpose.

---

## Out-of-domain data

**Social media vs news domain** — models trained on news articles perform poorly on social media text. Sentence structure, vocabulary, and register differ significantly. Metric scores will reflect this domain shift and should not be interpreted as model failure without accounting for it.

**Unseen entities** — translation models cannot reliably translate named entities they were not exposed to during training. If a person, organisation, or place name rose to prominence after the training cut-off, or is specific to a region where the model's training data was sparse, it may be:
- Transliterated incorrectly
- Left in the source script
- Dropped entirely

This has a direct downstream impact on NER and topic modelling. If NER runs on translated text, missed or corrupted entity spans will produce gaps in the entity extraction. For topic modelling, cross-language clustering of the same entity depends on the entity name appearing consistently in both languages — if the model translates it differently (or not at all) across documents, the same entity will not cluster together.

**Code switching** — text that mixes two languages (common in Indonesian, Tagalog, Arabic social media, sometimes Chinese) affects language detection and model performance. Most models are not trained on code-switched text and will produce degraded output on mixed-language sentences. See [Yong et al. (2023)](https://arxiv.org/abs/2311.12405) for a survey of code-switching challenges in LLMs.

**Multi-lingual text** — documents where more than one language appears (beyond code switching — e.g. multilingual reports, parallel content in one file) affect language detection at the document level and may also degrade model performance if the dominant language is misidentified.

---

## Language-specific challenges

### Contextual inference

Korean, Chinese, and Japanese frequently omit subjects and other grammatical elements when the referent is clear from prior context. At sentence level, this context is often not available. The model may produce an ambiguous or incorrect translation (defaulting to a wrong pronoun, for example) because the source sentence alone does not contain enough information to determine who is performing the action.

### Idiomatic language

Idioms rarely translate literally. A model that is not aware of an idiom will translate its words, not its meaning, producing a semantically incorrect output that may score poorly on METEOR but reasonably on BERTScore (since the words are translated correctly even if the meaning is not).

**Example — "Kill two birds with one stone" across languages:**

| Language | Idiom | Literal translation |
|---|---|---|
| Spanish | Matar dos pájaros de un tiro | Kill two birds with one shot |
| French | Faire d'une pierre deux coups | Hit two flies with one stone |
| German | Zwei Fliegen mit einer Klappe schlagen | Hit two flies with one swat |
| Italian | Prendere due piccioni con una fava | Take two pigeons with one bean |
| Chinese | 一石二鸟 | One stone two birds |

**Language-specific idioms with no English equivalent:**

| Language | Idiom | Literal | Meaning |
|---|---|---|---|
| Japanese | 石の上にも三年 | Three years on a rock | Persistence pays off |
| Arabic | قلبه في ماء الورد | His heart is in rosewater | He is in good spirits |
| Mandarin Chinese | 狐假虎威 | The fox borrows the tiger's fierceness | Bullying others by flaunting powerful connections |

A model unaware of these will translate literally. "Three years on a rock" has a completely different semantic meaning from "Persistence pays off" — and their METEOR and BERTScore against a correct translation will reflect that.

Chinese idioms (Chengyu) are always 4 characters but can translate to entire sentences in English, producing large sequence length discrepancies that affect tokenisation and score calibration.

### Language scripts and parsing

**Right-to-left scripts** — Arabic, Farsi, Hebrew, Urdu read right-to-left. Pre-trained models handle this internally, but it is important not to assume left-to-right layout when processing or displaying text programmatically.

**Punctuation differences** — French, German, Spanish use additional punctuation not present in English (« », ¿). Chinese and Japanese use different full stops (。) and commas (，). **Critically: Chinese, Japanese, and Thai do not use spaces between words.** Sentences are written as continuous strings.

This has a direct practical consequence: splitting text into sentences by splitting on full stops (`.`) will not work for Chinese, Japanese, or Thai, because these languages use `。` as a sentence boundary. Any pre-processing pipeline that assumes Latin-style sentence boundaries will fail to segment these languages correctly, which affects the quality of the text passed to the translation model.

### Language resources

Low-resource languages present a compounded challenge: fewer pre-trained translation models, fewer benchmark evaluation datasets, and lower quality evaluation data where it exists. For these languages:
- Many-to-many models (mbart, madlad400) may be the only available option
- Benchmark datasets may cover only formal or news-domain text
- Evaluation conclusions should be held more lightly — the benchmark may not represent the project data at all
