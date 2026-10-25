# Neuro-Symbolic Causal Inference

Extract cause→effect relations from text, build a causal graph (DAG), fit a Bayesian network to real survey data, and answer
"what if we intervened?" questions. A rule-based extractor (spaCy) is compared with a fine-tuned transformer (DistilBERT).

Everything is in one notebook: **`causal_nlp.ipynb`**. Its first cell has a table of contents and a summary of results.

## Setup (Python 3.14)

```bash
uv venv .venv --python 3.14
uv pip install --python .venv/bin/python -r requirements.txt
.venv/bin/python -m ipykernel install --user --name python3 --display-name "Python 3 (.venv)"
```

Plain `pip` works too: `python3.14 -m venv .venv && .venv/bin/pip install -r requirements.txt`.

`requirements.txt` lists the direct dependencies and reads `requirements-lock.txt`, which pins **every** package, including indirect
ones. Keep both files. Pinning only the direct packages is not enough: newer releases of spaCy's helper packages (`preshed`, `cymem`)
break loading its English model.

## Run

Open `causal_nlp.ipynb` with the **Python 3 (.venv)** kernel and run all cells, or from the terminal:

```bash
.venv/bin/jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=-1 causal_nlp.ipynb
```

With the cached files below, a full run takes about 6 minutes on a laptop. Without `models/`, DistilBERT is retrained first,
which takes about 20 minutes on an Apple M-series GPU and longer on CPU.

## What is in the folder

| Path | What it is | Can it be regenerated? |
|---|---|---|
| `causal_nlp.ipynb` | The deliverable notebook | – |
| `requirements.txt`, `requirements-lock.txt` | Exact package versions used (direct + full lock) | – |
| `data/pubmed/abstracts.jsonl` | 500 PubMed abstracts (smoking & lung disease) | **Keep it.** PubMed search results change over time, and the labels in `data/labels/` belong to these exact abstracts. The notebook stops with an error if they no longer match. |
| `data/labels/` | Blind samples and Claude's silver labels (150 SemEval, 300 PubMed) | **Keep it.** Made once by an instruction model (Claude); the notebook cannot regenerate it. |
| `data/review/edges_*.csv` | Every extracted graph edge with an example sentence, plus a `keep` column | Rewritten on each run. **Reviewer decisions are kept.** |
| `data/nhanes/` | NHANES 2013–2018 survey files + linked mortality | Yes, downloaded automatically. *Not in the git repo.* |
| `models/distilbert-causal/` | Fine-tuned DistilBERT (best epoch) | Yes, by retraining: set `RETRAIN = True` in Step 5. *Not in the git repo* (255 MB, over GitHub's file limit), so after cloning, the first run trains it (~20 min). |
| `figures/` | DAG images | Yes, on every run |

SemEval-2010 Task 8 is downloaded from the Hugging Face hub on each run (`SemEvalWorkshop/sem_eval_2010_task_8`).

## Reviewing edges (team task)

The brief asks that only reviewed predictions reach the graph. Open `data/review/edges_rules.csv` (and `edges_distilbert.csv`), read
the example sentence for each edge, and type `yes` or `no` in the `keep` column. Re-run the notebook: edges marked `no` are removed,
and the notebook reports how many edges are still unreviewed.

## Data sources

- SemEval-2010 Task 8: https://huggingface.co/datasets/SemEvalWorkshop/sem_eval_2010_task_8
- PubMed E-utilities API: https://eutils.ncbi.nlm.nih.gov/entrez/eutils/
- NHANES: https://wwwn.cdc.gov/nchs/nhanes/ and the NCHS linked mortality files: https://ftp.cdc.gov/pub/Health_Statistics/NCHS/datalinkage/linked_mortality/
- Reference conclusions for graph evaluation: US Surgeon General's reports, 2014 (https://www.ncbi.nlm.nih.gov/books/NBK294322/, https://www.ncbi.nlm.nih.gov/books/NBK294300/) and 2010 (https://www.ncbi.nlm.nih.gov/books/NBK53021/)
