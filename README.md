# TakeMeter — Classifying r/leagueoflegends Discourse

A text classifier that sorts posts and comments from **r/leagueoflegends** into three
kinds of discourse: **`hot_take`**, **`analysis`**, and **`rant`**. The project compares a
zero-shot LLM baseline (Groq `llama-3.3-70b-versatile`) against a fine-tuned
`distilbert-base-uncased` model on the same held-out test set.

**Headline result:** the zero-shot baseline (accuracy **0.84**, macro-F1 **0.83**) clearly
beat the fine-tuned model (accuracy **0.55**, macro-F1 **0.42**). Fine-tuning on ~141
examples overfit and collapsed toward the majority class — a negative result discussed in
depth below.

---

## Community & Task

**Community:** r/leagueoflegends, focused on champion/meta discussion — patch megathreads,
"is X broken?" posts, tier-list threads, and daily discussion comments.

**Why it fits a classification task:** League's two-week patch cycle produces text-heavy
discourse whose *quality and intent* vary enormously. Regulars themselves distinguish
"cope" (emotional venting) from "based" (bold but defensible takes) from "actual analysis"
(data-backed argument). Those community-recognized distinctions are exactly what the labels
capture, which makes the boundaries meaningful rather than arbitrary.

---

## Labels

| Label | Definition |
|---|---|
| `hot_take` | A bold, confident opinion about a champion or the meta stated **without** supporting evidence — asserts a claim rather than arguing for it. |
| `analysis` | A structured claim backed by **specific, verifiable evidence** — win rates, patch-note changes, item interactions, or pro-play observations. The evidence must hold up the claim. |
| `rant` | An **emotional reaction** to a frustrating experience — venting, feeling-driven, evidence-free. If removing the frustration leaves nothing debatable, it's a rant. |

**Key decision rule (used for annotation and prompting):** *judge by substance, not tone.*
An analytical tone doesn't make a post `analysis` (it needs evidence); emotional framing
doesn't make a post `rant` (if a debatable claim remains after removing the emotion, it's a
`hot_take`). Full taxonomy, decision rules, and documented hard edge cases are in
[planning.md](planning.md).

> **Note on scope change:** the original spec planned 4 labels including `meta_report`. It
> was dropped to 3 because neutral meta-report posts were too rare to collect in real data
> (see Spec Reflection).

---

## Dataset

- **202 labeled examples**, all real posts/comments collected from r/leagueoflegends
  ([dataset.csv](dataset.csv)).
- Single CSV (not pre-split); the notebook does a 70/15/15 train/val/test split
  (~141 / 30 / 31).
- Columns: `text`, `label`, `source_url`, `pre_labeled`, `notes`.

| Label | Count | Share |
|---|---|---|
| hot_take | 84 | 42% |
| analysis | 64 | 32% |
| rant | 54 | 27% |

No class exceeds 70%. The set is imbalanced toward `hot_take` because confident opinions
are simply the most common post type on the subreddit; standalone `analysis` and pure
`rant` are rarer (see Spec Reflection). `pre_labeled` tracks provenance: `TRUE` = AI
pre-labeled then human-reviewed, `FALSE` = labeled directly by me (see AI Usage).

---

## Method

- **Baseline:** Groq `llama-3.3-70b-versatile`, zero-shot, `temperature=0`, one example per
  label in the prompt. Output constrained to a single label string. 31/31 responses
  parseable (0% unparseable).
- **Fine-tuned:** `distilbert-base-uncased`, 3 epochs (the reported "Run 1"), evaluated on
  the identical 31-example test split.

---

## Evaluation Report

### Overall

| Model | Accuracy | Macro-F1 |
|---|---|---|
| Baseline (Groq zero-shot) | **0.839** | **0.83** |
| Fine-tuned (DistilBERT) | 0.548 | 0.42 |
| Δ | **−0.29** | **−0.41** |

The fine-tuned model performed **worse across the board**. This is unusual but explainable
(see Reflection + the negative-result write-up in [finetuned_results.md](finetuned_results.md)).

### Per-class metrics

**Baseline (Groq):**
| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 1.00 | 0.60 | 0.75 | 10 |
| hot_take | 0.86 | 0.92 | 0.89 | 13 |
| rant | 0.73 | 1.00 | 0.84 | 8 |

**Fine-tuned (DistilBERT):**
| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 0.83 | 0.50 | 0.62 | 10 |
| hot_take | 0.48 | 0.92 | 0.63 | 13 |
| rant | **0.00** | **0.00** | **0.00** | 8 |

The standout: the fine-tuned model **never correctly predicts `rant`** (F1 0.00), while the
baseline handled it well (F1 0.84).

### Fine-tuned confusion matrix (test set)

Rows = true label, columns = predicted label. (Also saved as `confusion_matrix.png`.)

| True ↓ / Pred → | analysis | hot_take | rant |
|---|---|---|---|
| **analysis** (10) | 5 | 5 | 0 |
| **hot_take** (13) | 1 | 12 | 0 |
| **rant** (8) | 0 | 8 | 0 |

**The pattern is overwhelming and directional:** 13 of 14 errors are *something* →
`hot_take`. The model predicts `hot_take` 25 of 31 times and predicts `rant` **zero**
times. It collapsed toward the majority class.

### AI-assisted pattern analysis (then verified by hand)

I pasted all 14 misclassifications into an LLM and asked it to surface common themes. It
proposed:
1. **Majority-class collapse** — 13/14 errors land on `hot_take`.
2. **`rant` fully absorbed** — all 8 test rants → `hot_take`.
3. **Near-random confidence** — every prediction sits at ~0.37–0.39 (3-class chance = 0.33),
   i.e. the model barely discriminates.
4. **Length/numbers as an `analysis` proxy** — the single reverse error (hot_take→analysis)
   is the longest, most numeric post.

**Verification + what I discarded:** I re-read each example to confirm. Patterns 1–3 hold
cleanly. Pattern 4 is suggestive but rests on a single example, so I treat it as a
hypothesis, not a finding. I also considered and **discarded** a "sarcasm causes errors"
theory — the errors span sarcastic and earnest posts roughly evenly, so sarcasm isn't the
driver; majority-class collapse is.

### 3 errors analyzed in depth

**Error 1 — `analysis` → `hot_take` (evidence ignored).**
> *"Hard to understand how 56% win rate Senna emerald+ couldn't be hot fixed for 2 weeks but rather just 'ban her'."* — true `analysis`, predicted `hot_take` (conf 0.37)

- **Which boundary:** analysis→hot_take (5 of 14 errors).
- **Why it's hard:** the post cites a real stat (56% win rate) — the signal for `analysis` —
  but wraps it in a short, frustrated rhetorical complaint that *reads* like an opinion. The
  model keys on surface form, not the embedded evidence.
- **Labeling or data problem:** data. I labeled it consistently (the win-rate figure is the
  basis of the argument). With only ~45 `analysis` training examples — most of them dry and
  clinical — the model learned "analysis = technical-sounding," not "analysis = contains
  evidence." Casually-phrased, stat-bearing posts are underrepresented.
- **Fix:** more `analysis` examples where evidence is embedded in casual or emotional
  phrasing, so the model decouples "has evidence" from "sounds clinical."

**Error 2 — `rant` → `hot_take` (the dead class).**
> *"The most frustrating thing in lane for me is having to play around a gimmick… Maybe I'm just bad… but I can't stand having to play into champs like Illaoi…"* — true `rant`, predicted `hot_take` (conf 0.37)

- **Which boundary:** rant→hot_take (8 of 14 errors — *every* test rant).
- **Why it's hard:** `rant` and `hot_take` are both negative and opinionated; this post's
  core ("characters that force you to play around a gimmick") sounds like a debatable claim,
  and nuance like the self-deprecating "maybe I'm just bad" is invisible at this data scale.
- **Labeling or data problem:** data/boundary. With ~38 `rant` training examples, the model
  never carves out a `rant` region of feature space — it folds it entirely into `hot_take`.
  This is the single largest driver of the score drop.
- **Fix:** many more `rant` examples, and possibly a sharper definition that hard-separates
  rant from hot_take (e.g., "rant contains *no* debatable claim whatsoever").

**Error 3 — `hot_take` → `analysis` (the one reverse error).**
> *"Junglers deserve an anti-invade advantage… the invading champion would take increased damage while dealing reduced damage until 2:30 or 3:30…"* — true `hot_take`, predicted `analysis` (conf 0.37)

- **Which boundary:** hot_take→analysis (1 of 14 errors).
- **Why it's hard:** it's a long, structured design *proposal* packed with specific numbers
  (2:30, damage modifiers) that superficially resemble evidence — but it's speculation about
  a change that doesn't exist, not evidence about the current game.
- **Labeling or data problem:** boundary/definition. The `analysis` definition ("specific,
  verifiable evidence… item interactions") can be gamed by a detailed hypothetical; the
  model appears to read "numbers + length → analysis."
- **Fix:** tighten the `analysis` definition to distinguish *evidence about the current
  state* from *detailed speculation/proposal*, and add training examples of long, numeric,
  evidence-free proposals labeled `hot_take`.

### Sample Classifications

Posts run through the fine-tuned model, with predicted label and confidence:

| Post (truncated) | True | Predicted | Confidence |
|---|---|---|---|
| "Hard to understand how 56% win rate Senna emerald+ couldn't be hot fixed…" | analysis | hot_take | 0.37 |
| "The most frustrating thing in lane… I can't stand having to play into champs like Illaoi" | rant | hot_take | 0.37 |
| "Junglers deserve an anti-invade advantage… increased damage until 2:30" | hot_take | analysis | 0.37 |
| ⚠️ *[correct example — fill from notebook]* | hot_take | hot_take | *[fill]* |
| ⚠️ *[correct example — fill from notebook]* | analysis | analysis | *[fill]* |

> ⚠️ **TODO before submitting:** the notebook only printed *wrong* predictions, so the two
> correct rows above are placeholders. Run this snippet in the eval section and paste two
> real correctly-classified examples + confidences:
> ```python
> correct_idx = np.where(ft_pred_ids == ft_true_ids)[0]
> for idx in correct_idx[:6]:
>     print(f"{ID_TO_LABEL[ft_true_ids[idx]]} (conf {ft_probs[idx][ft_pred_ids[idx]]:.2f}): {test_df.iloc[idx]['text'][:110]}")
> ```

**Why a correct prediction is reasonable:** the fine-tuned model's strongest class is
`hot_take` (recall 0.92) — when it correctly tags a blunt, evidence-free opinion like
*"I think Faker should retire"* as `hot_take`, that's reasonable: the post asserts a
debatable claim with no supporting evidence and no emotional-venting framing, which is the
prototypical center of the `hot_take` class the model learned best.

---

## Reflection: What the Model Captured vs. What I Intended

**Intended:** a three-way distinction along *function* — does the post supply evidence
(`analysis`), assert a claim without evidence (`hot_take`), or just vent emotion with no
debatable claim (`rant`)?

**What the model actually captured:** essentially a **binary "is this a confident
opinion?"** detector that defaults to `hot_take`, plus a weak secondary "is this long and
full of numbers?" signal that occasionally flips to `analysis`. It **never learned `rant`
at all** — that region of meaning simply doesn't exist in its decision boundary.

**What it overfit to:** *surface form* — post length, presence of numbers, and opinionated
phrasing — rather than the semantic function my labels are built on. Ironically, the
guiding principle I wrote for human annotators ("judge by substance, not tone") is exactly
what the model failed to do: it judged by tone and form because, with ~141 examples, it
never got enough signal to learn substance.

**What it missed:** the entire emotional-venting class, and any `analysis` whose evidence is
phrased casually. The gap between my definitions (functional) and the model's boundary
(formal) is the core story of this project — and it's *why* the 70B baseline, with vastly
more world knowledge, succeeded where the small fine-tune could not.

---

## Definition of Success — Did We Hit It?

planning.md set these bars. Honest scorecard:

| Criterion | Target | Fine-tuned result | Met? |
|---|---|---|---|
| Macro-F1 | ≥ 0.75 | 0.42 | ❌ |
| No class below | 0.65 F1 | rant = 0.00 | ❌ |
| Beat the Groq baseline | ≥ baseline macro-F1 | 0.42 vs 0.83 | ❌ |
| `analysis` precision | ≥ 0.70 | 0.83 | ✅ |

The fine-tuned model met only the `analysis`-precision sub-goal (it's conservative about
calling things analysis, so when it does, it's usually right). It failed every headline
bar. The **baseline**, by contrast, would have cleared the macro-F1 and no-class-below bars.
The honest conclusion: for this task and data size, the zero-shot 70B model *is* the
deployable tool; the from-scratch fine-tune is not.

---

## Spec Reflection

**One way the spec helped:** the documented decision rules and hard edge cases made the
results immediately interpretable. I predicted in planning.md that the `hot_take↔analysis`
and `rant↔hot_take` boundaries would be the failure points — and the confusion matrix
landed exactly there. Committing to **macro-F1 + per-class + confusion matrix** (rather than
accuracy alone) in the eval plan is what surfaced the `rant` 0.00 collapse; plain accuracy
(0.55) would have hidden a totally dead class.

**One way the implementation diverged:** the spec planned **4 balanced labels (≈50 each,
including `meta_report`)**; the implementation shipped **3 imbalanced labels (84/64/54)**. I
dropped `meta_report` because neutral, opinion-free meta summaries were too rare to collect
without fabricating them, and I accepted real-but-imbalanced data over balanced-but-
synthetic data (I tried generating rants to balance the set and **discarded** them — see AI
Usage). That imbalance directly fed the majority-class collapse — a real cost of the
divergence, and an honest tradeoff: I prioritized data authenticity over class balance.

---

## AI Usage

I used AI tools (Claude, and Groq for the baseline/pre-labeling) at several stages. All
final labeling decisions were mine.

1. **Annotation pre-labeling (disclosed).** I directed an LLM to suggest a label for each
   collected post using my planning.md definitions; I then reviewed and corrected every
   suggestion. The `pre_labeled` column records this: `TRUE` = AI-suggested-then-reviewed,
   `FALSE` = labeled directly by me. The LLM proposed; I decided.

2. **Data cleaning / structuring.** I pasted raw copied Reddit text (messy, multi-post
   blocks) into Claude, which parsed it into CSV rows, fixed quoting, flagged duplicates,
   and noted borderline cases in a `notes` column. I made the final call on every label and
   on which borderline posts to keep.

3. **Discarded synthetic data (disclosed for integrity).** To fix the thin `rant` class, I
   had an LLM generate ~40 rant examples. On review I recognized they were synthetic —
   templated, generic, no real sources — which violated the real-data requirement and my own
   planning.md. I **removed all 40** and replaced them with **20 real collected rants**
   (each with a `source_url`). Nothing generated remains in the dataset.

4. **Wrong-prediction pattern surfacing.** I gave the 14 misclassifications to an LLM to
   surface themes (majority-class collapse, rant absorption, length-as-analysis). I verified
   each against the examples myself and **discarded** the "sarcasm" hypothesis as
   unsupported (see Evaluation Report).

5. **Prompt and spec drafting.** Claude helped draft the Groq classification/pre-label
   prompts ([prompts.md](prompts.md)) and structure planning.md sections; I edited the
   definitions, decision rules, and success thresholds to match my own judgment.

---

## Repository Contents

| File | What it is |
|---|---|
| [planning.md](planning.md) | Spec: community, labels, decision rules, hard edge cases, eval plan, AI plan |
| [dataset.csv](dataset.csv) | 202 labeled examples |
| [prompts.md](prompts.md) | Groq stress-test, pre-label, and classification prompts |
| [baseline_results.md](baseline_results.md) | Baseline numbers + reconstructed confusion matrix |
| [finetuned_results.md](finetuned_results.md) | Fine-tuned results, both runs, overfitting analysis |
| `evaluation_results.json` | Machine-readable baseline vs fine-tuned summary |
| `confusion_matrix.png` | Fine-tuned confusion matrix image |

## Reproduction

1. Open the Colab notebook, set runtime to **T4 GPU**.
2. Add `GROQ_API_KEY` via Colab Secrets (🔑) — **never commit the key**.
3. Upload `dataset.csv`. Set `LABEL_MAP = {"analysis":0, "hot_take":1, "rant":2}`.
4. Run baseline → train (DistilBERT) → evaluate. The notebook writes
   `evaluation_results.json` and `confusion_matrix.png` to download into this repo.
