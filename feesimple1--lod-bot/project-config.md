---
trigger: always_on
description: This is an automation engine for the non-player bot flowcharts in **Liberty or Death** (GMT Games, COIN Series Vol. V). It handles 0–4 bot-controlled factions (British, Patriots, Indians, French) via an interactive CLI. The goal is faithful implementation of the published flowcharts — not a variation, not a reinterpretation, not a "simplified" version.
---

# CLAUDE.md — Liberty or Death Bot Engine

## Project Summary

This is an automation engine for the non-player bot flowcharts in **Liberty or Death** (GMT Games, COIN Series Vol. V). It handles 0–4 bot-controlled factions (British, Patriots, Indians, French) via an interactive CLI. The goal is faithful implementation of the published flowcharts — not a variation, not a reinterpretation, not a "simplified" version.

**Language:** Python (100%). ~43,000 LOC across ~130 files.
**Tests:** pytest. 1,136 tests currently passing.
**No external game references.** Do not consult BoardGameGeek, other COIN games, other GMT titles, or any historical sources outside the Reference Documents.

---

## Source of Truth (Strict Hierarchy)

These are **read-only**. Never modify them. All other code must conform to them.

### 1. `lod_ai/rules_consts.py` — Canonical Labels
Every string label for factions, pieces, markers, leaders, and space IDs used anywhere in the codebase **must** come from this file. If a label doesn't exist here, it is wrong.

**Canonical piece tags:**
```
British_Regular, British_Tory, British_Fort, British_Regular_Unavailable, British_Tory_Unavailable
Patriot_Continental, Patriot_Militia_A, Patriot_Militia_U, Patriot_Fort
French_Regular, French_Regular_Unavailable
Indian_WP_A, Indian_WP_U
Village, Squadron, Blockade, Propaganda, Raid
```

**Faction strings:** `BRITISH`, `PATRIOTS`, `INDIANS`, `FRENCH`

**If you encounter a string literal in the code that doesn't match one of these** (e.g., `"Continental"`, `"Militia"`, `"British_cube"`, `"Tory"` used as a piece tag, `"fort"`, etc.), it is a bug. Replace it with the correct constant from `rules_consts.py`.

### 2. `Reference Documents/card reference full.txt` — Card Behavior
The authoritative definition of every card's unshaded and shaded effects. Card handler implementations in `lod_ai/cards/effects/` must match this file exactly — same targets, same piece types, same destinations ("to Casualties" vs "to Available" vs "remove"), same conditions.

### 3. `Reference Documents/` — Everything Else
All files in the `Reference Documents/` directory are source-of-truth materials. Always check the full directory contents rather than relying solely on this list. Key files include but are not limited to:

- `*bot flowchart and reference.txt` — Non-player decision trees (one per faction)
- `Manual Ch 1.txt` through `Manual Ch 8.txt` — Full rules
- `Manual Glossary.txt` — Definitions
- `leader_capabilities.txt` — Leader special abilities
- `1775 Scenario Reference.txt`, `1776 Scenario Reference.txt`, `1778 Scenario Reference.txt` — Setup data
- `map_base.csv` — Map topology (CSV; the only non-.txt reference file)
- `setup instructions.txt` — Scenario setup procedure
- `random spaces map.txt`, `random spaces table.txt` — Random space selection tables

If a file exists in `Reference Documents/` and is not listed above, it is still authoritative. Any file added to this directory in the future is automatically a source of truth.

All files are plain text (except `map_base.csv`). No PDFs, no images.

---

## Lessons learned (for future COIN-bot projects)

`coin-implementation-lessons.md` in the repo root distills ~40+ bugs
encountered in this project into 11 anti-pattern families, with
architecture/testing/process recommendations.  Audience is future
projects implementing other COIN-series games (or any tabletop
simulation with bot AI).  Not part of the runtime; not a code
reference; safe to ignore unless you're starting a similar project.

---

## Supplementary strategic reference (not a source of truth)

`strategy.md` in the repo root is a faction-by-faction, scenario-by-
scenario strategy guide for human (or LLM) gameplay decisions.  It is
**not** part of the runtime, **not** authoritative for rules questions,
and **not** a substitute for the bot flowchart references in
`Reference Documents/`.  Bot implementations must follow the published
flowcharts, not strategic intuitions from this guide.

Use it when you need to understand *why* a faction prefers one play
over another — for example, to author better commit messages, investigate
bot-flowchart anomalies, or write human-side CLI prompts.  See the file's
own "What this file is and is NOT" section.

---

## Project Structure

```
lod_ai/
├── __init__.py, __main__.py     # Entry point: python -m lod_ai
├── rules_consts.py              # SOURCE OF TRUTH — canonical labels
├── engine.py                    # Main game loop / turn sequencing
├── interactive_cli.py           # Human player menus
├── cli_utils.py                 # CLI helper functions
├── dispatcher.py                # Routes faction turns to bot or human
├── victory.py                   # Victory condition checks
├── bots/                        # Non-player bot logic (flowcharts)
│   ├── base_bot.py              # Shared bot infrastructure
│   ├── british_bot.py           # British flowchart implementation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FeeSimple1/lod-bot](https://github.com/FeeSimple1/lod-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
