# TakeMeter – Project 3 Planning
## Milestone 1: Community, Labels, and Taxonomy Design

---

## Community

**r/leagueoflegends** — champion and meta discussion threads.

The League of Legends subreddit is one of the most active gaming communities online, with text-heavy posts ranging from emotional reactions to patch changes all the way to data-backed champion analysis. The champion/meta discussion space is especially rich because the game changes every two weeks (patch cycle), which means discourse quality varies wildly: some posts cite real statistics and patch notes, others are pure venting or unsubstantiated hot takes. These distinctions are meaningful to regulars — the community actively calls out "low-effort" takes vs. posts with real evidence.

---

## Label Taxonomy

### `hot_take`
**Definition:** A bold, confident opinion about a champion or the meta stated without supporting evidence. The post asserts a claim rather than arguing for it.

**Clear examples:**
- *"Yasuo is completely broken right now, he wins every matchup in the game."*
- *"Riot keeps buffing ADCs because they don't know how to balance the game."*

**Borderline example:**
- *"Jinx is S-tier right now, her win rate is insane."* — mentions a stat loosely, but doesn't cite it or build an argument from it.

---

### `analysis`
**Definition:** Makes a structured claim about a champion or the meta backed by specific, verifiable evidence — win rates, patch note changes, item interactions, or pro play observations.

**Clear examples:**
- *"Jinx's win rate jumped 3% after 14.5 because her Q cooldown reduction now synergizes with Infinity Edge's crit scaling — here's the math."*
- *"Thresh has a 52.3% win rate in Plat+ since the support item changes in 14.4, up from 49.1%. The extra gold generation lets him spike earlier."*

**Borderline example:**
- *"Zed is broken, his kill pressure at level 6 with one item is statistically the highest of any assassin."* — cites a stat, but the framing is accusatory and no source is given.

---

### `rant`
**Definition:** An emotional reaction to a frustrating in-game experience. Feeling-driven and evidence-free — the post is venting, not arguing.

**Clear examples:**
- *"I'm so sick of playing against Zed, there is literally nothing you can do once he gets one item."*
- *"Lost 10 games in a row because of inting supports, this game is actually unplayable."*

**Borderline example:**
- *"I hate how Riot keeps buffing Yasuo every patch — it feels like they just want him to be strong forever."* — emotional framing, but references Riot's patch behavior, which edges toward hot_take.

---

### `meta_report`
**Definition:** A neutral summary of the current state of the meta — tier lists, "what's strong right now," or patch rundowns — without a strong personal opinion or emotional charge.

**Clear examples:**
- *"Support tier list after patch 14.6: S tier — Thresh, Nautilus; A tier — Lulu, Karma; B tier — Soraka."*
- *"Quick patch 14.7 breakdown: Jinx buffed (Q cd reduced), Zed nerfed (R cooldown up), Grasp of the Undying unchanged."*

**Borderline example:**
- *"Here's my personal tier list — I think Thresh is overrated and Lulu is slept on."* — has tier list structure but introduces personal opinion, which edges toward hot_take.

---

## Decision Rules for Edge Cases

### Hot_take vs. Analysis
**The key question:** Would the evidence hold up the claim even if you removed the opinion framing?

- If a post cites a specific, verifiable stat (e.g., "52.3% win rate in Plat+") **and** builds reasoning around it → `analysis`
- If a stat is mentioned loosely or decoratively — just enough to sound credible but without being the basis of an argument → `hot_take`
- When in doubt: accusatory or emotional framing is a signal toward `hot_take` even if a number appears

### Rant vs. Hot_take
**The key question:** Is the post primarily expressing a feeling, or making a claim?

- If you could remove the frustration and still have a debatable opinion → `hot_take`
- If removing the frustration leaves nothing → `rant`

### Meta_report vs. Hot_take
**The key question:** Is the post describing the meta, or arguing about it?

- Tier lists with personal justifications or contrarian framing → `hot_take`
- Tier lists presented as informational summaries → `meta_report`

---

## Why These Distinctions Matter

In the League of Legends community, the difference between a rant, a hot take, and real analysis determines whether a post gets engagement or gets dismissed. Regulars actively distinguish "cope" (emotional, evidence-free) from "based" (bold but defensible) from "actual analysis" (data-backed). These labels capture that spectrum in a way that is consistent across annotators and meaningful to the community itself.

---

## Hardest Anticipated Edge Case

A post like: *"Jinx is broken — her win rate is the highest it's been all season."*

This could be `hot_take` (no source, accusatory framing) or `analysis` (references a real stat). Decision rule: if the stat is vague, unsourced, or used as punctuation rather than as the basis of reasoning, label it `hot_take`. A genuine `analysis` post would specify the number, the patch context, and explain *why* the stat means what the poster claims.

---

## Milestone 2: Spec

---

## Data Collection Plan

**Where:** Posts and top-level comments from r/leagueoflegends, focused on champion/meta discussion threads — patch-day megathreads, "is X broken?" posts, tier-list posts, and the daily/weekly discussion threads. Comments are valuable here because rants and hot takes show up more in replies than in standalone posts.

**How many:** A balanced set of **50 examples per label × 4 labels = 200 total**. Balanced classes keep accuracy interpretable and give the fine-tuned model enough signal per class on a small dataset.

**Collection method:** Manual browsing + copy into a CSV (`text`, `label`, `source_url`, `pre_labeled` columns). I'll pull from a spread of recent patches rather than a single thread, so the model doesn't overfit to one champion or one patch's vocabulary.

**If a label is underrepresented after 200:** `analysis` is the label most at risk of falling short, since data-backed posts are rarer than venting. If any label lands under 50:
1. **Targeted search** for that label specifically — e.g. for `analysis`, search threads with "win rate," "patch notes," "data," or link-heavy posts; for `meta_report`, search "tier list" and patch-rundown posts.
2. If still short, **rebalance the target** down to the size of the smallest class (e.g. 40/label) rather than padding one class with weak or borderline examples — class balance matters more than hitting exactly 200.
3. I will **not** synthetically generate real training examples; AI is only used for stress-testing definitions and pre-label suggestions I review (see AI Tool Plan).

---

## Evaluation Metrics

**Primary: Macro-averaged F1.** Because the task has 4 classes and the boundary cases (`hot_take` vs `analysis`) are exactly where the model is most likely to fail, I need a metric that weights every class equally regardless of how common it is. Macro-F1 averages per-class F1, so a model that nails the easy `rant`/`meta_report` classes but collapses `analysis` and `hot_take` together will score poorly — which is the correct signal.

**Secondary: Per-class precision and recall.** Accuracy alone hides *which* class is failing and *how*. Precision/recall per class tells me, for example, whether `analysis` is being under-predicted (low recall — the model rarely commits to it) or over-predicted (low precision — it labels loose hot takes as analysis). That directly informs whether my label definitions or my data need work.

**Diagnostic: Confusion matrix.** This shows the *direction* of errors. My hypothesis is that the dominant confusion will be `hot_take ↔ analysis` and `rant ↔ hot_take`, mirroring the edge cases above. The confusion matrix lets me confirm or refute that and target failure analysis accordingly.

**Why not accuracy alone:** With balanced classes accuracy isn't useless, but it can't distinguish a model that's uniformly decent from one that's excellent on 3 classes and broken on the 4th. For a tool meant to separate "real analysis" from "cope," failing one class quietly is a real failure, so per-class metrics are required.

---

## Definition of Success

**Quantitative bar:** A **macro-F1 ≥ 0.75** on the held-out test set, with **no single class scoring below 0.65 F1.** The second clause matters: a high macro-average that hides one broken class would not be a useful community tool.

**Relative bar:** The fine-tuned DistilBERT should **match or beat the Groq (llama-3.3-70b) baseline on macro-F1.** If a from-scratch fine-tune on 200 examples can't reach a large general-purpose LLM, that's a meaningful negative result worth reporting honestly.

**"Good enough for deployment" reasoning:** For a real community tool that flags low-effort takes vs. evidence-backed posts, the costly error is mislabeling genuine `analysis` as `hot_take` (penalizing good contributors). So beyond the macro bar, I want **`analysis` precision ≥ 0.70** — when the tool says something is analysis, it should usually be right. These thresholds are specific enough that at the end I can objectively check each one against the evaluation output and declare pass/fail.

---

## AI Tool Plan

**1. Label stress-testing (before annotating).**
Before collecting examples, I'll give an LLM (Groq, llama-3.3-70b) my four label definitions and the edge-case decision rules, and ask it to generate 8–10 posts that deliberately sit on the `hot_take ↔ analysis` and `rant ↔ hot_take` boundaries. If I can't classify its outputs cleanly using my own rules, that's a signal my definitions are too loose — I'll tighten them *before* annotating 200 examples, not after. Outcome of this step will be noted in the AI usage write-up.

**2. Annotation assistance (pre-labeling with review).**
I will use **Groq (llama-3.3-70b) to pre-label** each example, then review every pre-labeled row myself and correct it — the LLM proposes, I decide. The final label is always my human judgment. For disclosure and integrity:
- The CSV includes a **`pre_labeled` column** (`TRUE`/`FALSE`) marking every row that received an AI suggestion.
- I'll also track **how often I overrode the AI's suggestion**, which doubles as a sanity check on my own label definitions (a high override rate on one class means that definition is ambiguous).
- The AI usage section of the README will disclose the tool, the prompt, and the override rate.

**3. Failure analysis (after evaluation).**
After evaluation, I'll give an LLM the list of misclassified examples (text, true label, predicted label) and ask it to identify **patterns** in the errors — e.g. "the model labels any post containing a number as `analysis`," or "short emotional posts get split between `rant` and `hot_take`." I will then **verify each proposed pattern myself** by going back to the actual misclassified posts and confirming the pattern holds before writing it into my evaluation — the AI surfaces hypotheses, I confirm them against the data. I expect the patterns to cluster around the `hot_take ↔ analysis` boundary predicted by the confusion matrix.
