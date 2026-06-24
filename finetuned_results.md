# Fine-Tuned Results — DistilBERT (distilbert-base-uncased)

Recorded 2026-06-23. Evaluated on the **same 31-example test split** used for the
baseline, so the two are directly comparable. See [baseline_results.md](baseline_results.md).

## Headline
The fine-tuned model **underperformed the zero-shot baseline and largely failed to
learn** — it collapsed to predicting the majority class (`hot_take`).

| Metric | Baseline (Groq) | Fine-tuned (DistilBERT) |
|---|---|---|
| Accuracy | 0.839 | **0.548** |
| Macro-F1 | 0.83 | **0.42** |
| analysis F1 | 0.75 | 0.62 |
| hot_take F1 | 0.89 | 0.63 |
| rant F1 | 0.84 | **0.00** |

## Per-class (fine-tuned)
| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 0.83 | 0.50 | 0.62 | 10 |
| hot_take | 0.48 | 0.92 | 0.63 | 13 |
| rant | 0.00 | 0.00 | 0.00 | 8 |

## Confusion matrix (from confusion_matrix.png)
Rows = true, columns = predicted.

| True ↓ / Pred → | analysis | hot_take | rant |
|---|---|---|---|
| **analysis** (10) | 5 | 5 | 0 |
| **hot_take** (13) | 1 | 12 | 0 |
| **rant** (8) | 0 | 8 | 0 |

- The model predicts `hot_take` 25/31 times and **never predicts `rant`** (rant column = 0).
- All prediction confidences cluster at **~0.37–0.39** (3-class random = 0.33), i.e. the
  model barely discriminates — a signature of majority-class collapse / undertraining.

## Hypothesis outcome
The pre-registered hypothesis (analysis recall and rant precision would *rise* after
fine-tuning) was **refuted**. The failure was not the predicted tone-confusion at the
`analysis`/`hot_take` boundary but a more fundamental **majority-class collapse**: the
`rant` class, which the baseline handled well (F1 0.84), was lost entirely (F1 0.00).

## Three errors analyzed (for README)
1. **rant → hot_take (the dead class).**
   *"The most frustrating thing in lane for me is having to play around a gimmick… I can't
   stand having to play into champs like Illaoi."* — clear venting, predicted hot_take
   (conf 0.37). The model never learned `rant` at all; all 8 rants became hot_take. This is
   the single largest driver of the accuracy drop.

2. **analysis → hot_take (evidence ignored).**
   *"Hard to understand how 56% win rate Senna emerald+ couldn't be hot fixed…"* — explicit
   stat (56%), predicted hot_take (conf 0.37). The baseline used numeric cues to find
   `analysis`; the fine-tuned model ignores them and defaults to hot_take.

3. **hot_take → analysis (the one reverse error).**
   *"Junglers deserve an anti-invade advantage… increased damage while dealing reduced
   damage until 2:30…"* — a design *proposal* with no evidence (labeled hot_take), predicted
   analysis (conf 0.37). When the model does deviate from hot_take, it appears to key on
   length/specificity as a proxy for "analysis" rather than actual evidence.

Together: the model **defaulted to the majority class** and, on rare deviation, used
**surface length rather than substance** — the opposite of the "judge by substance, not
tone" principle from planning.md.

## Remediation attempt — Run 2 (10 epochs), then reverted
I suspected undertraining / class imbalance, so I retrained for 10 epochs. **It got worse,
which is itself the diagnostic answer**, so Run 1 remains the reported model.

Run 2 test accuracy: **0.452** (down from Run 1's 0.548). `rant` F1 stayed **0.00**.

Run 2 training curve (the key evidence — **overfitting, not a bug**):
| Epoch | Train Loss | Val Loss | Val Acc |
|---|---|---|---|
| 5 | 0.90 | 0.91 | 0.67 (best) |
| 7 | 0.58 | 0.82 | 0.63 |
| 10 | 0.27 | 0.84 | 0.60 |

Training loss fell cleanly (1.04 → 0.27) while validation loss bottomed at epoch 7 and
then rose — the model memorized the ~141 training examples and stopped generalizing. More
epochs *lowered* test accuracy. This rules out "undertraining" and confirms the small-data
ceiling.

## Conclusion
Two runs confirm a **genuine small-data ceiling**, not a fixable training bug:
- Training converged (loss → 0.27), so the pipeline works.
- More epochs *lowered* test accuracy (overfitting), so undertraining wasn't the issue.
- **`rant` collapses to 0.00 F1 in BOTH runs** — at ~38 rant training examples, DistilBERT
  cannot separate emotional venting (`rant`) from emotional opinion (`hot_take`).
- The same examples (#2 Senna 56%, #3 Hullbreaker, #4 perfect storm, #9 Dblade) fail in
  both runs — genuinely hard, not noise.

A from-scratch DistilBERT fine-tuned on ~141 examples **cannot match the 70B zero-shot
baseline** (0.84). This is the meaningful negative result anticipated in planning.md.

**Reported model:** Run 1 (3-epoch config), accuracy 0.548 — the better fine-tuned run,
matching `evaluation_results.json`.

## Caveat
Test set is only 31 examples (analysis n=10, rant n=8). Treat the *patterns* (rant
collapse, hot_take↔analysis confusion, overfitting curve) as the signal, not exact decimals.
