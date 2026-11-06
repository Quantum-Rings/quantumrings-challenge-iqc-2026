# Submission Harness

Predict how long each circuit takes to simulate, straight from the `.qasm` file.
This harness produces the **one file you send back to us**.

## Setup (once)

```bash
pip install -r requirements.txt
```

## What you edit

**Only `model.py`.** Implement these methods:

| Method | Job | Called |
|---|---|---|
| `RuntimeModel()` | load your trained model | once |
| `featurize(qasm_text)` | parse a circuit into features | once per circuit |
| `predict(features, threshold)` | return **predicted seconds** (one number) | once per (circuit, threshold) |

There is no timeout flag — if you think a run will time out, just predict a duration
**≥ the 6-hour cap (21600 s)**.

A trivial baseline is already in `model.py` so the harness runs before you touch
anything. Replace its body with your real parser and model.

**Caps:** `featurize` and `predict` must each run in **≤ 15 s per circuit**. The
harness times you and warns on anything over.

## What's included

These live at the top level of the repo, next to `submission-harness/`:

| Path | Contents |
|---|---|
| `../train-circuits/` | the training circuits, one `<id>.qasm.zst` per circuit |
| `../train-runtime-data.csv` | training labels, one row per `(circuit, threshold)` run |

The circuits are `zstd`-compressed. `run.py` decompresses them for you, so you
don't need raw files to run the harness. To get raw `.qasm` files for exploring
and training, see
[Getting the raw `.qasm` files](../README.md#getting-the-raw-qasm-files) in the
main README. Decompress them **outside** `train-circuits/`, or the harness will
read each circuit twice.

## Run it

1. In the final hours we push the hold-out circuits to this repo, in a
   `holdout-circuits/` folder at the top level next to `train-circuits/`.
   Run `git pull` to get them.
2. Generate your submission:

```bash
python run.py --team "Your Team Name" --circuits ../holdout-circuits
```

Without `--circuits`, `run.py` predicts the training circuits in
`../train-circuits/`, which is handy for checking your score (below).

This writes **`submission.csv`**:

```
team,filename,threshold,pred_duration_s,parse_s,predict_s
```

3. **DM `submission.csv` back to us.** That's your entry.

## Check your score first (optional but recommended)

You have the training labels (`../train-runtime-data.csv`). Run `run.py` on the
training circuits (the default), then score yourself with the *exact* metric we use:

```bash
python run.py --team "Your Team Name"
python score.py --pred submission.csv --labels ../train-runtime-data.csv
```

**Tip:** scoring on circuits your model trained on overstates how it will do on
the unseen hold-out. Set aside a validation set to estimate that score, then
retrain on all the training data for your final submission. Note that
`score.py` scores every label and counts a missing prediction as 0, so to score
a validation set, filter the labels to those circuits or compute the metric
yourself. See [Reference scores](../README.md#reference-scores) in the main
README for scores to compare against.

## How the automated third is scored

Your total score is **1/3 automated + 2/3 subjective** (presentation, novelty,
process, etc. — judged live, five equally-weighted categories). The automated third
is **pure duration accuracy**, per `(circuit, threshold)`:

```
score = max(0, 1 − |log10(pred / actual)|)      # exact = 1.0, off by 10× = 0
```

Runtimes span seconds to hours, so accuracy is measured in **log scale** — being
2× off costs the same whether the run is 1 second or 1 hour. Timeout circuits are
scored against the 6-hour cap, so predicting ≥ cap earns full credit on them and
predicting seconds for one is penalized like any other large miss.

Your score is the average over **every** hold-out run. A run missing from your
`submission.csv` scores 0, so make sure every circuit gets a prediction.

## Rules

- Build your own parser; no starter feature list is provided.
- You must be able to explain every part of your submission.
- Don't try to recover the source algorithm from filenames/metadata.
