# Baseline Results — Groq Zero-Shot (llama-3.3-70b-versatile)

Recorded 2026-06-23. These are the **baseline** numbers (no fine-tuning yet),
to be compared against the fine-tuned DistilBERT on the *same* test set.

## Setup
- **Model:** Groq `llama-3.3-70b-versatile`, zero-shot (one example per label in the system prompt)
- **Temperature:** 0 (deterministic)
- **Test set:** 31 examples (the notebook's 15% test split of the 202-example dataset)
- **Parseable responses:** 31 / 31 (0% unparseable — prompt format clean, no revision needed)

## Overall
- **Accuracy:** 0.839
- **Macro avg F1:** 0.83
- **Weighted avg F1:** 0.83

## Per-class
| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 1.00 | 0.60 | 0.75 | 10 |
| hot_take | 0.86 | 0.92 | 0.89 | 13 |
| rant | 0.73 | 1.00 | 0.84 | 8 |

## Reconstructed confusion matrix
(Derived from the per-class precision/recall; rows = true, columns = predicted.)

| True ↓ / Pred → | analysis | hot_take | rant |
|---|---|---|---|
| **analysis** (10) | 6 | 2 | 2 |
| **hot_take** (13) | 0 | 12 | 1 |
| **rant** (8) | 0 | 0 | 8 |

## Hypothesis (to test after fine-tuning)
The baseline **under-predicts `analysis` (recall 0.60)** and uses **`rant` as a
catch-all (precision 0.73)**. It appears to classify by *surface tone rather than
substance*: evidence-backed posts with emotional framing get called `rant`
(2 analysis → rant), and evidence-backed posts that sound like confident opinions
get called `hot_take` (2 analysis → hot_take). `analysis` has perfect precision but
poor recall — the model only commits to `analysis` when a post *looks* clinical, and
misses analysis wrapped in opinion or frustration. No `rant` or `hot_take` was ever
misread as `analysis` (the model is conservative about that label).

This matches the `hot_take ↔ analysis` boundary predicted in planning.md, plus the
`rant ↔ analysis` hard edge case documented there (frustrated tone, real argument).

**Prediction:** if fine-tuning teaches substance-over-tone, `analysis` recall and
`rant` precision should both rise, with `hot_take` staying strong.

## Caveat
The test set is only 31 examples (analysis n=10, rant n=8). One or two flips move
these numbers substantially, so treat the *pattern* (analysis under-recall, rant
over-predict) as the signal, not the exact decimals.
