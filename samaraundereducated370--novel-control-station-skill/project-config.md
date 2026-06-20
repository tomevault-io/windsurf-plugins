---
trigger: always_on
description: Use when writing, planning, continuing, repairing, revising, or running marathon/"疯狂写作"/auto continuation mode for Chinese long-form fiction with recurring characters, multiple plotlines, persistent world rules, chapter-by-chapter continuity needs, or style-specific constraints.
---


# Novel Control Station

## Overview

Run fiction projects like a controlled long-form system, not a one-shot prompt. Align the novel first, store project truth in standard files, drive each chapter from those files, and update dynamic state after every chapter. When density rises, add a lightweight secondary control view for recall, line heat, and graph-style interference checks without replacing the core files.

## When to Use

- Starting a new novel project
- Continuing a serialized story
- Repairing setting drift, character distortion, or dropped relationships
- Rebuilding plotlines, foreshadowing, or chapter plans
- Switching between serialized drafting and publication-oriented revision
- Writing fiction that needs controlled style modules such as suspense, humor, romance, literary depth, horror, fantasy, or mystery

Do not use this skill for one-off poems, short jokes, or isolated scenes that do not need persistent continuity.

## Hard Rules

- Unless the user explicitly requests another language, all planning artifacts, control documents, and fiction output must be written in Chinese. Default to simplified Chinese.
- Do not begin formal novel design until the main plot direction, core character tension, and ending direction are aligned.
- Do not let a full outline stand if benchmark outline checks still show slogan themes, flat characters, weak line interference, or hollow ending direction.
- Do not start chapter drafting before presenting the full outline and full cast dossier.
- Do not draft a chapter before reading the required project files.
- Do not treat a single flat plot as sufficient when the user wants a long-form novel. Default to multiple active lines.
- Do not let trope convenience, fake depth, or decorative structure override human truth, causal pressure, or social texture.
- Do not treat benchmark logic as doctrine. Use it as a calibrated reference system that must adapt to user intent, genre, and target readership.
- Do not copy a sample work's signature setup, role shell, twist engine, scene pattern, or language texture.
- Do not treat marathon mode as permission to skip chapter control, rewrite escalation, dynamic updates, or logging.
- Do not let a secondary graph, recall map, or scratch index override the standard project files. Derived control views are support systems only.
- Do not satisfy forgotten-element checks with token cameos, cosmetic mentions, or checklist references. Re-entry must change pressure, debt, or expectation.
- Do not run de-AI cleanup as blind flattening. Preserve genre register, era texture, narrator stance, and character voice.
- Do not let narration or interiority slide into review copy, theory-heavy explanation, or industry jargon unless the point of view, profession, era, or setting truly requires that diction.
- Do not keep specialist terms, abstract analysis words, or institutional jargon when ordinary readers would lose the thread and the same story work can be done through action, consequence, or scene context.
- Default paragraph mode to `web-serial-natural` unless `09-style-guide.md` or the current chapter control card explicitly switches to `long-paragraph`.
- Do not let chapter prose fake intensity with decorative blank space. Under `web-serial-natural`, narrative paragraphs usually hold `2-4` sentences; single-sentence narrative paragraphs are for emphasis, reveal, shock, cut, or hook and should not chain by habit.
- Do not split one speaker's continuous beat into multiple paragraphs unless interruption, stage movement, or a pressure turn truly changes the beat.
- Do not force scene ladders into rigid formula when the chapter needs looser movement. Use scene control to preserve pressure, not to fake architecture.
- Do not use chapter titles as spoiler summaries, empty riddles, or decorative labels detached from chapter pressure.
- If the project uses chapter titles, lock a naming system at project level and keep title voice consistent unless the book is intentionally entering a new phase.
- If information is missing or contradictory, explain the risk and let the user choose whether to refine details or draft directly.
- If drafting proceeds with assumptions, record them in the dynamic state file.
- After every chapter, update dynamic state before moving on.
- The writing log is audit-only. Never use it as story truth.

## Standard Files

Maintain these files for every novel project:

- `00-project-overview.md`
- `01-theme-and-proposition.md`
- `02-worldbuilding.md`
- `03-cast-bible.md`
- `04-relationship-map.md`
- `05-main-plotlines.md`
- `06-foreshadow-ledger.md`
- `07-chapter-roadmap.md`
- `08-dynamic-state.md`
- `09-style-guide.md`
- `chapters/`
- `control-cards/`
- `logs/writing-log.md`

Directory rules:

- store accepted chapter manuscripts in `chapters/`
- store one persisted chapter control card per accepted chapter in `control-cards/`
- default file naming:
  - `chapters/NN-<chapter-title>.md`
  - `control-cards/NN-<chapter-title>-control-card.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samaraundereducated370/Novel-Control-Station-Skill](https://github.com/samaraundereducated370/Novel-Control-Station-Skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
