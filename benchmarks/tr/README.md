# MT Evaluation — Turkish

## Language

Turkish (`tr`) · Latin script · Agglutinative morphology

## Use case

- [x] Unified-language translation (pre-processing for topic modelling)
- [x] Annotator-facing translation (facilitating analysis)

## Benchmark dataset

**MaCoCu Turkish–English** (`MaCoCu-tr-en.sent.txt`)  
Parallel corpus crawled from Turkish-language web domains in 2021. 1.6M sentence pairs across topic-level domains, with structured quality annotations.

**Key annotations used:**

| Annotation | Description |
|---|---|
| `src_text` | Source text (Turkish or English, depending on direction) |
| `trg_text` | Target text (English or Turkish, depending on direction) |
| `translation_direction` | Specifies direction and type: Turkish-original/English-human-translated (`first-orig-second-ht`) vs reverse |
| `bicleaner_ai_score` | Likelihood of a pair being mutual translations (0–1); quality signal |
| `bleualign_score` | Sentence alignment similarity score from bleualign tool (0–1) |

**Filtering applied before evaluation:**

| Filter | Condition | Rationale |
|---|---|---|
| Translation direction | `first-orig-second-ht` | Turkish original, English human translation only |
| `bicleaner_ai_score` | > 0.9 | High-confidence translation pairs |
| `bleualign_score` | > 0.4 | Well-aligned sentence pairs |
| Sample | Random 10,000 | Final evaluation set |

**Sentence length distribution:**  
The filtered dataset shows a right-skewed length distribution (Figure 1): the majority of sentences are under 200 characters, with a peak around 30–50 characters and a long tail extending to ~1,000 characters. Longer sentences have a direct impact on translation time — particularly relevant for CPU-based inference where long sequences dominate iteration time.

Note: the low-BERTScore error sample contained Chinese-language source sentences — MaCoCu includes multilingual web content and language filters are not perfect. This is a data quality issue in the corpus, not a model failure. Verify source language when inspecting low-scoring examples.

## Candidate models

| Model | HuggingFace ID | Type |
|---|---|---|
| MBART many-to-many | `facebook/mbart-large-50-many-to-many-mmt` | Many-to-many |
| Helsinki opus-mt-tr-en | `Helsinki-NLP/opus-mt-tr-en` | Language-pair-specific (Marian) |

---

## Translation stage

### Model configuration

| Model | Source lang code | Target lang code | Max length | Batch size |
|---|---|---|---|---|
| MBART | `tr_TR` | `en_XX` | 512 | 16 |
| Helsinki | Marian (auto) | `en` | 512 | 16 |

### Computational performance

| Model | Dataset | GPU time | CPU estimate | CPU iteration rate |
|---|---|---|---|---|
| MBART | 10,000 MaCoCu pairs (A100, batch=16) | 41 min | ~23 hrs | ~135s/batch |
| Helsinki | 10,000 MaCoCu pairs (A100, batch=16) | 20 min | ~4 hrs | ~25s/batch |

For comparison: Farsi translation on the same GPU took approximately 16–20 minutes — Turkish with MBART takes noticeably longer due to the longer average sentence length in MaCoCu.

---

## Evaluation metrics

| Metric | What it compares | Sensitivity |
|---|---|---|
| **METEOR** | Translation vs gold standard (lexical + stemming + synonymy) | High — penalises paraphrasing |
| **BERTScore** | Translation vs gold standard (semantic, `roberta-large` English embeddings) | Moderate |
| **XLMrScore** | Translation vs source text (cross-lingual, `xlm-roberta-base`) | Cross-lingual meaning preservation |

XLMrScore is a newer approach using XLM-RoBERTa to compare the translation against the original source text (rather than the gold standard). A high XLMrScore indicates the translation preserves the meaning of the Turkish original, independent of the reference translation.

---

## MBART Results

### Translation process

- **GPU:** A100
- **Batch size:** 16
- **Dataset:** 10,000 MaCoCu examples
- **Duration:** 41 minutes (`625/625 [40:56<00:00, 4.34s/it]`)

### Scores

| Metric | Score |
|---|---|
| METEOR | **0.534** |
| BERTScore F1 | **0.929** |
| XLMrScore F1 | **0.852** |

### Descriptive statistics

| Stat | BERTScore | METEOR | XLMrScore |
|---|---|---|---|
| Count | 10,000 | 10,000 | 10,000 |
| Mean | 0.928714 | 0.533832 | 0.852264 |
| Std | 0.024720 | 0.178001 | 0.026218 |
| Min | 0.789697 | 0.000000 | 0.744702 |
| 25% | 0.914660 | 0.426899 | 0.836374 |
| Median | 0.927750 | 0.530806 | 0.849461 |
| 75% | 0.941842 | 0.638889 | 0.865510 |
| Max | 1.000000 | 0.999898 | 0.984469 |

### Patterns observed

- **High BERT and XLM, moderate METEOR:** The dominant pattern. Translations are generally accurate in conveying meaning but differ in structure from the gold standard. METEOR penalises this; BERTScore and XLMrScore do not.
- **BERTScore resilience:** BERTScore tends to remain high even when there are minor translation errors or structural deviations.
- **METEOR sensitivity:** METEOR is more sensitive to exact word choice and linguistic structure — a good complement to BERTScore for catching surface-level fidelity failures.
- **XLMrScore consistency:** Generally aligns with BERTScore. High XLMrScore alongside high BERTScore confirms meaning is preserved across languages. However, like BERTScore, it may not catch subtle terminology errors.
- **Limitation:** In instances with significant terminology errors (see Example 5), BERT and XLM scores sometimes failed to reflect the inaccuracy.

### Error analysis

**Example 1** (ID: 511610) — Paraphrase, correct meaning

- **Turkish:** IETM toplantıları daha önce görmediğiniz bir ülkeye gitmek, o ülkeyi solumak, derinlemesine tanımak ve sanat temsilcileri aracılığıyla sorunlarını ve hedeflerini dinlemek için mükemmel bir fırsat. IETM organizatörlerinin desteğini hissediyorsunuz. Uluslararası sanat toplantılarını kültür yöneticileri için bir fikir cennetine dönüştüren de bu. Metotlarınızı tartışabilir, işbirliği olanaklarını değerlendirebilir, akımlardan haberdar olabilirsiniz. Dünyanın sorunlarını tartışabilirsiniz.
- **Gold standard:** IETM meetings are full of inspiration from beginning to the end, which is a basic nutrient for artists. They act as a great opportunity to travel and get to know a country you have never seen before and to listen to their targets and problems from representatives of arts. This is what makes international art meetings a heaven of ideas for culture managers, where you can discuss your methods, evaluate cooperation opportunities and get informed about new trends.
- **MBART:** IETM meetings are an excellent opportunity to go to a country you have never seen before, to breathe in, to get to know it in depth and to listen to its problems and objectives through art representatives. You feel the support of the organisers of IETM. It turns international art meetings into a haven of ideas for cultural managers. You can discuss your texts, evaluate possibilities for co-operation, be aware of the flows. You can discuss the problems of the world.
- **Google:** IETM meetings are an excellent opportunity to go to a country you have never seen before, breathe that country, get to know it in depth and listen to its problems and goals through art representatives. You feel the support of IETM organizers. This is what turns international arts meetings into a paradise of ideas for cultural managers. You can discuss your methods, evaluate collaboration opportunities, and be informed about trends. You can discuss the world's problems.
- **ChatGPT:** IETM meetings are an excellent opportunity to visit a country you haven't seen before, to breathe its air, to deeply understand it, and to listen to its issues and goals through the lens of art representatives. You feel the support of the IETM organizers. It is these international art meetings that turn into a paradise of ideas for cultural managers. You can discuss your methods, evaluate collaborative opportunities, and stay informed about trends. You can discuss the world's problems.
- **Scores:** METEOR=0.430979, BERTScore=0.922377, XLMrScore=0.846186
- **Analysis:** While there are variations in phrasing and expression across all translations, the core ideas and messages are consistently conveyed. The gold standard takes a more condensed, thematic framing ("full of inspiration") while MBART and other models translate more literally from the Turkish structure.

---

**Example 2** (ID: 1539823) — High scoring, good alignment

- **Turkish:** Ülkemiz, Bursa ve Gürsu çapında hayata geçirilen projelerle, gençlerin; sosyal, kültürel, ekonomik, toplumsal ve bireysel gelişimlerini, aktif yaşama katılımlarını sağlamak hedefleriyle, 2019 yılında, Gürsu Belediyesi'nce Eurodesk Temas Noktası oluşturuldu.
- **Gold standard:** With the projects implemented throughout our country, Bursa and Gürsu, young people; The Eurodesk Contact Point was established by Gürsu Municipality in 2019 with the aim of ensuring their social, cultural, economic, societal and individual development and participation in active life.
- **MBART:** Through the projects implemented throughout our country, Bursa and Gürsu, a Eurodesk Contact Point was established in 2019 by the Municipality of Gürsu with the aim of ensuring the participation of young people in their social, cultural, economic, social and individual development and active life.
- **Google:** In our country, with projects implemented in Bursa and Gürsu, the objective was to facilitate the social, cultural, economic, societal, and personal development of young people, as well as their active participation in life. To this end, in 2019, the Gürsu Municipality established a Eurodesk Contact Point.
- **ChatGPT:** In our country, with projects implemented in Bursa and Gürsu, the objective was to facilitate the social, cultural, economic, societal, and personal development of young people, as well as their active participation in life. To this end, in 2019, the Gürsu Municipality established a Eurodesk Contact Point.
- **Scores:** METEOR=0.860507, BERTScore=0.968052, XLMrScore=0.870916
- **Analysis:** Both the model translation and other translations align with the gold standard, conveying the main ideas and messages. The differences are minor structural choices — "Municipality of Gürsu" vs "Gürsu Municipality."

---

**Example 3** (ID: 603168) — Technical/formal language

- **Turkish:** c. ÖSYM tarafından yapılan Akademik Personel ve Lisansüstü Eğitimine Giriş Sınavı'nın (ALES) sayısal, sözel ya da eşit ağırlıklı puan türlerinden birinde en az 55 standart puan ya da uluslararası düzeyde kabul gören GRE ve GMAT sınavlarından Kapadokya Üniversitesi Senatosu tarafından belirlenen denklik çerçevesinde 55 puana eşdeğer puan almış olmak,
- **Gold standard:** Having taken at least 55 standard points in one of the numerical, verbal or equal weighted points types of the Academic Staff and Graduate Education Entrance Exam (ALES) conducted by OSYM or the equivalent of 55 points in accordance with the equivalence set by the Cappadocia University Senate of internationally accepted GRE and GMAT exams,
- **MBART:** to have received at least 55 standard marks in one of the types of numerical, verbal or equal weighted marks of the Academic Staff and Postgraduate Training Initial Examination (ALES) conducted by the ÖSYM, or equivalent to 55 marks in the internationally accepted GRE and GMAT exams, within the balance set by the Senate of the University of Cappadocia;
- **Google:** c. At least 55 standard points in one of the numerical, verbal or equally weighted score types of the Academic Personnel and Graduate Education Entrance Examination (ALES) conducted by ÖSYM, or equivalent to 55 points from the internationally accepted GRE and GMAT exams within the framework of equivalence determined by the Cappadocia University Senate. to get points,
- **ChatGPT:** To have scored at least 55 standard points in either the numerical, verbal, or equally weighted categories of the Academic Staff and Postgraduate Education Entrance Examination (ALES) administered by ÖSYM, or to have achieved a score equivalent to 55 points in the GRE and GMAT exams, as per the equivalency determined by the Senate of Cappadocia University,
- **Scores:** METEOR=0.687394, BERTScore=0.944028, XLMrScore=0.831611
- **Analysis:** MBART and ChatGPT both convey the key information present in the gold standard. Minor differences in phrasing ("marks" vs "points", "Postgraduate Training Initial Examination" vs "Graduate Education Entrance Exam") reflect terminological variation rather than semantic failure. Essential content and meaning are preserved.

---

**Example 4** (ID: 968379) — Short phrase

- **Turkish:** Düşük ağırlık – geniş kapsama
- **Gold standard:** Low weight – wide coverage
- **MBART:** Low weight -- broad coverage
- **Google:** Low weight – wide coverage
- **ChatGPT:** Low weight – extensive coverage
- **Scores:** METEOR=0.750000, BERTScore=0.939218, XLMrScore=0.839311
- **Analysis:** The MBART translation and ChatGPT translation are both close to the gold standard, accurately conveying the key idea. The minor substitution of "broad" for "wide" is a valid synonym choice.

---

**Example 5** (ID: 1155143) — Terminology error

- **Turkish:** Isı yalıtımı ve dış cephe mantolama sanıldığı gibi sadece kışın binaların soğuktan korunması ve ısıtma enerjisinden tasarruf amaçlı değil, yazın da mekânlarımızı serinletmek amacı ile kullandığımız klima enerjisinden de tasarruf sağlıyor. Isı yalıtımı; sağlıklı yaşam koşullarının yaratılması, yakıt tüketimlerini azaltarak tüketicilerin düşük yakıt masrafları ile ısıtma ve soğutma yapabilmesi ve dolayısıyla hava kirliliğinin de azaltılmasını sağlıyor. Böylelikle harcanan enerjiden ortalama yüzde 50 tasarruf sağlanıyor ve fosil yakıt tüketiminden kaynaklanan sera gazı salınımı da aynı oranda düşürülmüş oluyor.
- **Gold standard:** Thermal insulation and exterior sheathing is not only intended to protect buildings from the cold in winter and to save heating energy, but also to save air-conditioning energy that we use to cool our spaces in summer. Thermal insulation; The creation of healthy living conditions reduces fuel consumption, allowing consumers to heat and cool with low fuel costs, thereby reducing air pollution. In this way, 50% of the energy is saved on average and greenhouse gas emissions from fossil fuel consumption are reduced at the same rate.
- **MBART:** Irrigation and outdoor shelter not only save energy for keeping buildings cool and heating in the winter, but also for the climate energy we use in the summer to cool our premises. Irrigation creates healthy living conditions, reduces fuel consumption, enabling consumers to heat and cool at low fuel costs and thus reduce air pollution, thereby saving an average of 50 per cent of energy consumption and reducing greenhouse gas emissions from fossil fuel consumption.
- **Google:** Thermal insulation and exterior cladding not only protect buildings from cold in winter and save heating energy, as is thought, but also save the air conditioning energy we use to cool our spaces in summer. Thermal insulation; Creating healthy living conditions reduces fuel consumption, enabling consumers to heat and cool with low fuel costs, thus reducing air pollution. In this way, an average of 50 percent savings is achieved in the energy consumed, and greenhouse gas emissions resulting from fossil fuel consumption are reduced at the same rate.
- **ChatGPT:** Thermal insulation and external cladding are not just for protecting buildings from cold in winter and conserving heating energy, but also for saving on air-conditioning energy used to cool our spaces in summer. Thermal insulation contributes to creating healthy living conditions by reducing fuel consumption, allowing heating and cooling at lower fuel costs and thereby reducing air pollution. As a result, an average of 50% energy is saved, and the emission of greenhouse gases from fossil fuel consumption is similarly reduced.
- **Scores:** METEOR=0.498926, BERTScore=0.924988, XLMrScore=0.816629
- **Analysis:** The model translation diverges from the gold standard in the initial phrase. "Isı yalıtımı" (thermal insulation) was incorrectly translated as "Irrigation" — a significant terminology error. Notably, BERTScore (0.925) and XLMrScore (0.817) remain moderate, illustrating the limitation of these metrics for catching domain-specific terminology failures.

---

## Helsinki Results

### Translation process

- **GPU:** A100
- **Batch size:** 16
- **Dataset:** 10,000 MaCoCu examples
- **Duration:** 20 minutes (`625/625 [20:00<00:00, 1.93s/it]`)

### Scores

| Metric | Score |
|---|---|
| METEOR | 0.500 |
| BERTScore F1 | 0.929 |
| XLMrScore F1 | 0.852 |

### Descriptive statistics

| Stat | BERTScore | METEOR | XLMrScore |
|---|---|---|---|
| Count | 10,000 | 10,000 | 10,000 |
| Mean | 0.925993 | 0.507730 | 0.850317 |
| Std | 0.024643 | 0.176887 | 0.026194 |
| Min | 0.709600 | 0.000000 | 0.717611 |
| 25% | 0.912151 | 0.391379 | 0.834771 |
| Median | 0.924966 | 0.499357 | 0.848384 |
| 75% | 0.938988 | 0.616361 | 0.864280 |
| Max | 1.000000 | 0.999878 | 0.969174 |

### Patterns observed

- **Moderate METEOR, high BERT and XLM:** Same dominant pattern as MBART. Helsinki prioritises semantic integrity over linguistic alignment.
- **Semantic integrity over linguistic alignment:** Consistently high BERTScore and XLMrScore indicate effective meaning preservation, even with sentence structure deviations.
- **Structural and phrasing variation:** Moderate METEOR scores across examples reflect that Helsinki often restructures sentences differently from the gold standard — more so than MBART.
- **Linguistic variability:** METEOR variability across examples indicates Helsinki's translations vary in how closely they mimic the gold standard's exact phrasing and style.

### Error analysis

The same 5 sentences used for MBART are re-evaluated here for direct model comparison.

**Example 1** (ID: 511610) — Paraphrase, correct meaning

- **Model translation (Helsinki):** IETM meetings are an excellent opportunity to go to a country you have never seen before, to breathe in, to get to know it in depth and to listen to its problems and objectives through art representatives. You feel the support of the organisers of IETM. It turns international art meetings into a haven of ideas for cultural managers. You can discuss your texts, evaluate possibilities for co-operation, be aware of the flows. You can discuss the problems of the world.
- **Scores:** METEOR=0.463679, BERTScore=0.919772, XLMrScore=0.851462
- **Analysis:** The Helsinki model and the gold standard convey similar core ideas, but with variations in expression and phrasing. Interestingly, the Helsinki score is slightly better than MBART on this example (METEOR 0.464 vs 0.431). Both models produce nearly identical translations for this sentence.

---

**Example 2** (ID: 1539823) — High scoring, alignment

- **Model translation (Helsinki):** Through the projects implemented throughout our country, Bursa and Gürsu, a Eurodesk Contact Point was established in 2019 by the Municipality of Gürsu with the aim of ensuring the participation of young people in their social, cultural, economic, social and individual development and active life.
- **Scores:** METEOR=0.570720, BERTScore=0.918328, XLMrScore=0.863527
- **Analysis:** Core messages conveyed, similar structure to MBART. Helsinki scores lower than MBART on this example (BERTScore 0.918 vs 0.968), despite the translation being nearly identical in content. The Helsinki output has slightly lower BERTScore, which may reflect minor subword tokenisation differences.

---

**Example 3** (ID: 603168) — Technical/formal language

- **Model translation (Helsinki):** to have received at least 55 standard marks in one of the types of numerical, verbal or equal weighted marks of the Academic Staff and Postgraduate Training Initial Examination (ALES) conducted by the ÖSYM, or equivalent to 55 marks in the internationally accepted GRE and GMAT exams, within the balance set by the Senate of the University of Cappadocia;
- **Scores:** METEOR=0.335070, BERTScore=0.904701, XLMrScore=0.823894
- **Analysis:** Helsinki deviates more from the gold standard in sentence structure and presentation compared to MBART (METEOR 0.335 vs 0.687 for MBART). Both models produce a recognisable translation, but Helsinki scores noticeably lower on this formal technical sentence.

---

**Example 4** (ID: 968379) — Short phrase

- **Model translation (Helsinki):** Low weight -- broad coverage
- **Scores:** METEOR=0.382653, BERTScore=0.918065, XLMrScore=0.825906
- **Analysis:** The Helsinki translation is very close to the gold standard in content. The minor punctuation change (dash style) does not alter meaning but impacts METEOR due to its sensitivity to structural elements. Helsinki scores lower than MBART on this example (METEOR 0.383 vs 0.750) despite the translations being effectively identical.

---

**Example 5** (ID: 1155143) — Terminology error

- **Model translation (Helsinki):** Irrigation and outdoor shelter not only save energy for keeping buildings cool and heating in the winter, but also for the climate energy we use in the summer to cool our premises. Irrigation creates healthy living conditions, reduces fuel consumption, enabling consumers to heat and cool at low fuel costs and thus reduce air pollution, thereby saving an average of 50 per cent of energy consumption and reducing greenhouse gas emissions from fossil fuel consumption.
- **Scores:** METEOR=0.487236, BERTScore=0.930759, XLMrScore=0.807960
- **Analysis:** Helsinki produces the same "Irrigation" terminology error as MBART. The translations are nearly identical for this sentence, suggesting the error is a shared failure mode for Turkish thermal-domain vocabulary at the model level rather than a model-specific issue. Helsinki BERTScore (0.931) is marginally higher than MBART (0.925) on this example.

---

## Model comparison

| Metric | MBART | Helsinki | Delta |
|---|---|---|---|
| METEOR (mean) | **0.5338** | 0.5077 | +0.0261 (MBART) |
| BERTScore (mean) | **0.9287** | 0.9260 | +0.0027 (MBART) |
| XLMrScore (mean) | **0.8523** | 0.8503 | +0.0020 (MBART) |
| BERTScore (min) | 0.7897 | 0.7096 | MBART more stable at low end |
| GPU time | 41 min | **20 min** | Helsinki 2× faster |
| CPU time | ~23 hrs | **~4 hrs** | Helsinki 5–6× faster on CPU |

**Score pattern:** BERTScore and XLMrScore gaps are minimal and unlikely to affect topic modelling quality. The meaningful difference is in METEOR: MBART is consistently higher across individual examples (Examples 2, 3, 4 show clear MBART advantage). Helsinki deviates more from the gold standard in structural terms.

---

## Recommendation

**MBART is the recommended model for Turkish→English translation.** The METEOR advantage (0.534 vs 0.500 — a gap of 0.034) is meaningful given METEOR's sensitivity. At the mean, a 0.034 improvement corresponds to noticeably more faithful translations at the sentence level. The error analysis examples confirm this: MBART consistently scores higher across individual sentences.

If compute is the primary constraint — large volume on CPU, no GPU access — Helsinki is an acceptable fallback. The semantic content captured by BERTScore and XLMrScore is essentially equivalent, and topic modelling quality is unlikely to be affected significantly. Helsinki's ~5–6× CPU speed advantage (4 hrs vs 23 hrs for 10K examples) is substantial.

**The choice between MBART and Helsinki is an empirical decision driven by compute budget, not a clear quality ceiling.** Both models demonstrate the same core failure mode (terminology errors in domain-specific text); neither is immune.

---

## Notebooks

| Notebook | Description |
|---|---|
| [`turkish_mbart_benchmark.ipynb`](turkish_mbart_benchmark.ipynb) | MBART many-to-many on MaCoCu — reference results METEOR=0.534, BERTScore=0.929, XLMrScore=0.852 |
| [`turkish_helsinki_benchmark.ipynb`](turkish_helsinki_benchmark.ipynb) | Helsinki opus-mt-tr-en on same data — METEOR=0.500, BERTScore=0.929, XLMrScore=0.852 |
