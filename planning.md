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
