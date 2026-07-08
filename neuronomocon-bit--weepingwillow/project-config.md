---
trigger: always_on
description: A horror/sci-fi novel series called **Weeping Willow**. First three books:
---

# WEEPING WILLOW — PROJECT STATUS & CONTEXT

## What This Project Is

A horror/sci-fi novel series called **Weeping Willow**. First three books:
- **Weeping Willow: The Absence** (Book 1)
- **Weeping Willow: The Hunger** (Book 2)
- **Weeping Willow: The Silence** (Book 3)

Series name is "Weeping Willow" — NOT "trilogy." Keeps the door open for future books.

## Workflow

- **Claude** handles: planning, scaffolding, series bible, chapter drafting (Book 2 onward), chapter review/audit, continuity tracking
- **Book 1** prose was drafted by ChatGPT. As of Book 2 (2026-05-14), ChatGPT was dropped — it could not reliably hold the prose hard rules — and **Claude drafts the chapters directly.** `series-bible/12-chatgpt-system-prompt.md` is retained as a consolidated rules reference.
- **Process:** Claude drafts a chapter into `chapter_review.md` → author proofreads/edits → on approval the final is saved to `chapters-book2/` and an "As Written" block is added to the brief.

## Series Bible (COMPLETE)

All 12 docs in `series-bible/`:

| File | Contents |
|------|----------|
| `01-world.md` | Setting (Lowport, Maine, ~2050), tech, Meridian origin, atmosphere, public awareness arc |
| `02-characters.md` | Full cast: Willow, Iris, Kade, Rourke, Caleb, Lena, Virek, Lila Mercer, Joel, Leah, Enzo, Joe E., Xander, Weston |
| `03-themes-and-tone.md` | Thematic pillars + tone/voice rules |
| `04-book1-outline.md` | "The Absence" — 3-act structure, detailed beats |
| `05-book1-chapter-briefs.md` | 22 chapters, full scene breakdowns + "As Written" blocks for completed chapters |
| `06-book2-outline.md` | "The Hunger" — act structure, origin reveal, Willow's choice |
| `07-book3-outline.md` | "The Silence" — final confrontation, destruction method, locked specifics |
| `08-book2-chapter-briefs.md` | 22 chapters, full scene breakdowns |
| `09-book3-chapter-briefs.md` | 22 chapters, full scene breakdowns |
| `10-subplot-threading.md` | 10 arcs tracked across all 66 chapters |
| `11-key-dialogue-notes.md` | Scene-level dialogue direction for ~15 key moments |
| `12-chatgpt-system-prompt.md` | Ready-to-paste system prompt for ChatGPT + drift corrections |
| `character-image-prompts.md` | DALL-E/ChatGPT image prompts — dark gothic painterly style, all 12 characters (16 prompts) |

## Writing Progress

**Chapters written and reviewed:**
- Book 1 Ch1 — The Gap ✅
- Book 1 Ch2 — Caleb Ward ✅
- Book 1 Ch3 — The First Case ✅
- Book 1 Ch4 — Lila Mercer ✅
- Book 1 Ch5 — Kade ✅
- Book 1 Ch6 — Rourke ✅

- Book 1 Ch7 — The Pattern ✅
- Book 1 Ch8 — Following the Thread ✅

- Book 1 Ch9 — The City Feels It ✅

- Book 1 Ch10 — Lena ✅

- Book 1 Ch11 — Caleb Shift ✅

- Book 1 Ch12 — The Taking of Caleb Ward (MIDPOINT) ✅

- Book 1 Ch13 — After ✅

- Book 1 Ch14 — The Weight ✅

- Book 1 Ch15 — Deterioration ✅

- Book 1 Ch16 — The Realization (ACT II TURN) ✅

- Book 1 Ch17 — Tracking ✅
- Book 1 Ch18 — The Deeper Zone ✅
- Book 1 Ch19 — Closing Distance ✅
- Book 1 Ch20 — Contact ✅
- Book 1 Ch21 — Partial Loss ✅
- Book 1 Ch22 — Exit ✅

**Book 1 — The Absence: COMPLETE (23,405 words / novella)**

**Book 1 — Full proofread/polish pass: COMPLETE (2026-04-04)**
- Question mark convention enforced: Iris/Taken characters use periods on questions (flat affect), warm characters (Rourke, Kade, Lena, Leah) use question marks. Pre-Taking Caleb Ch11 intentionally uses periods as foreshadowing.
- "Not X. Not Y. Z." pattern varied in densest stretch (Ch19-21)
- Duplicate phrase ("She held the contradiction") fixed in Ch15
- Fear-check formula trimmed from 5 to 2 full instances in Act III (Ch17 + Ch22 final); 3 middle instances varied
- Ch19 tightened (~90 lines consolidated)
- Ch7 As Written block reconciled with actual prose

**Book 1 — Deep audit pass: COMPLETE (2026-04-08)**
- 1 typo fixed (Ch1 "dim" → "Dim")
- Lila (Ch4) question mark corrected to period (Taken convention)
- Pre-Taking Caleb Ch11: 5 remaining question marks converted to periods (foreshadowing)
- Kade question marks fixed in Ch7 (3), Ch8 (3), Ch16 (1) — warm character convention
- Ch19 triplet density reduced: removed "Not inferred. Not concluded." (two full triplets within 6 lines → one)
- Ch22 duplicate phrasing varied: "The delay reduced. / Not removed. / Reduced." (identical to Ch21) → "The delay loosened. / Still present. / Less."
- Ch22 restored "alive" in "closer to Willow than anyone else alive" per brief
- Ch22 added missing brief beat: "She did not know which of those should frighten her."
- As Written blocks corrected: Ch20 head-tilt quote, Ch21 phantom negations removed, Ch19 device vibration attribution clarified, Ch22 updated to match new prose
- Individual chapter files saved in `chapters/` folder

**Book 1 — Chapter file review pass: COMPLETE (2026-04-10)**
Reviewing individual chapter files against briefs, audit notes, and conventions. Fixes applied to both `chapters/` files and `chapter_review.md`.
- Punctuation convention refined: full-sentence questions now use question marks for all characters (including flat-affect); short/tonal probes ("How." / "Why." / "Pain.") keep periods. Dialogue tags use "said" not "asked" for flat-affect characters.
- Device/terminal readouts reformatted: inline with colon + italic, not stacked on separate lines

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neuronomocon-bit/weepingwillow](https://github.com/neuronomocon-bit/weepingwillow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
