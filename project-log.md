# Evaluation Run Log

Running log of all translation evaluation runs. One row per model–dataset–language combination.

| Model | Language | Dataset | Samples | Speed | Dataset link | Notes | Output |
|---|---|---|---|---|---|---|---|
| `facebook/mbart-large-50-many-to-many-mmt` | — | opusTCv20210807+bt_transformer-big_2022-03-09.test.txt | 1250 | 2.21s/it (44:52 total) | — | Model handles many languages but weaknesses spotted with Spanish. | Detailed error analysis |
| `facebook/mbart-large-50-many-to-many-mmt` | Farsi (`fa_IR`) | MIZAN | 10K | 1.33s/it (16:53 total) | https://github.com/omidkashefi/Mizan | Gold standard quality is not ideal — looking into alternative resources before drawing conclusions. | Evaluation summary |
| `facebook/mbart-large-50-many-to-many-mmt` | Farsi (`fa_IR`) | PEPC | 10K | 2.06s/it (20:52 total) | https://iasbs.ac.ir/~ansari/nlp/pepc.html | Better gold standard than MIZAN — achieved stronger scores. Preferred benchmark for Farsi. | Evaluation summary |
