# Groq Prompts for TakeMeter

These prompts implement the AI Tool Plan from `planning.md`. Run them with the Groq
model `llama-3.3-70b-versatile` (the same one used for the baseline).

---

## 1. Label Stress-Testing Prompt (run BEFORE collecting)

Purpose: surface posts that sit on the boundary between two labels. If you can't
classify the outputs cleanly with your own decision rules, tighten the definitions
in `planning.md` before annotating real data.

```
You are helping me stress-test a text-classification label scheme for posts from
r/leagueoflegends. Here are my four labels and their definitions:

- hot_take: A bold, confident opinion about a champion or the meta stated without
  supporting evidence. The post asserts a claim rather than arguing for it.
- analysis: Makes a structured claim about a champion or the meta backed by specific,
  verifiable evidence — win rates, patch note changes, item interactions, or pro play
  observations.
- rant: An emotional reaction to a frustrating in-game experience. Feeling-driven and
  evidence-free — venting, not arguing.

Generate 10 short, realistic posts (1-3 sentences each, in the voice of real
r/leagueoflegends users) that deliberately sit on the BOUNDARY between two of these
labels and would be genuinely hard to classify. Focus especially on the
hot_take <-> analysis and rant <-> hot_take boundaries.

For each post, output:
1. The post text
2. The two labels it sits between
3. A one-sentence note on why it is ambiguous

Do NOT assign a single final label — the point is to find the hard cases.
```

After running this: for each ambiguous post, apply your decision rules. Any post you
still can't resolve means a definition needs tightening. Record what you changed.

---

## 2. Pre-Labeling Prompt (run on batches of REAL collected posts)

Purpose: get a first-pass label suggestion for posts you have already collected.
You MUST review and correct every suggestion yourself — the LLM proposes, you decide.
Mark every row it touched with `pre_labeled = TRUE` in the CSV.

```
You are labeling posts from r/leagueoflegends into exactly one of four categories.
Apply these definitions strictly:

- hot_take: A bold, confident opinion stated WITHOUT supporting evidence. Asserts a
  claim rather than arguing for it. Accusatory or emotional framing with a stray
  number still counts as hot_take if the number is decorative, not the basis of an
  argument.
- analysis: A structured claim backed by SPECIFIC, verifiable evidence (win rates with
  numbers, patch note changes, item interactions, pro play). The evidence must hold up
  the claim, not just decorate it.
- rant: An emotional reaction to a frustrating experience. Venting, evidence-free. If
  removing the frustration leaves nothing debatable, it's a rant.

For each numbered post below, output a line in the format:
<number>,<label>

Use only these label strings: hot_take, analysis, rant, meta_report.
Output nothing else — no explanations.

POSTS:
1. <paste post text>
2. <paste post text>
3. ...
```

After running this: paste the suggested labels into your CSV, then **read every post
and correct the label using your own judgment.** Track how often you override the
suggestion — a high override rate on one label means that definition is still
ambiguous. Disclose this workflow and the override rate in your README's AI usage
section.
