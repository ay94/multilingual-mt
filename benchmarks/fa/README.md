# MT Evaluation — Farsi (Persian)

## Language

Farsi / Persian (`fa`) · Arabic script · Right-to-left

## Benchmark datasets

Two parallel corpora evaluated — gold standard quality differs significantly between them.

**MIZAN**  
10,000 sentence pairs. Literary and formal register.  
Source: https://github.com/omidkashefi/Mizan  
Note: gold standard quality is inconsistent in places — scores should be interpreted cautiously until a cleaner reference dataset is confirmed.

**PEPC** (Persian–English Parallel Corpus)  
10,000 sentence pairs. Better gold standard than MIZAN — achieved stronger metric scores.  
Source: https://iasbs.ac.ir/~ansari/nlp/pepc.html  
Preferred benchmark for Farsi evaluation.

## Evaluation runs

| Dataset | Samples | Speed | Notes |
|---|---|---|---|
| MIZAN | 10K | 1.33s/it | Weak gold standard — inconclusive |
| PEPC | 10K | 2.06s/it | Better reference — stronger scores |

## Model

`facebook/mbart-large-50-many-to-many-mmt`  
Source language code: `fa_IR`  
Target language code: `en_XX`

## Language-specific notes

- Farsi uses Arabic script and reads right-to-left — display and parsing considerations apply
- BPE tokenisation on Arabic-script languages can produce long subword sequences; monitor truncation
- Literary-register evaluation data (MIZAN) may not represent social media or news domain well — gold standard domain matters as much as quality
- Code switching with English is common in social media Farsi and affects all models
