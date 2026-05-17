---
trigger: always_on
description: Generates pixel-accurate dwarf portraits from DF's own sprite sheets. Self-contained module at `src/df_storyteller/portraits/` — no dependencies on the rest of df_storyteller. Also published as standalone package [df-portrait-compositor](https://github.com/Been012/df-portrait-compositor).
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**df-storyteller** — A web-based storytelling companion for Dwarf Fortress. Captures game data via DFHack Lua scripts, parses legends XML exports, and generates AI-written narratives grounded in actual game events, dwarf personalities, and world history. Features a fantasy-themed web UI with live event feed, character sheets, and a world lore browser.

## Build & Run

```bash
# Install in development mode
pip install -e ".[dev]"

# One-time setup (prompts for DF path, LLM provider, API key)
python -m df_storyteller init

# Launch web UI (opens browser at localhost:8000)
python -m df_storyteller serve

# CLI commands (still work alongside web UI)
python -m df_storyteller status
python -m df_storyteller dwarves --detail
python -m df_storyteller chronicle
python -m df_storyteller bio "Urist"
python -m df_storyteller saga

# In DFHack console (first time per fortress):
# storyteller-begin              # Initial snapshot + start events
# storyteller-begin --yes        # Same + export legends
# storyteller-snapshot           # Manual snapshot (events auto-start on map load)
# storyteller-events status      # Check event monitoring
# storyteller-events debug       # Manual poll + show all dwarf state

# Run tests
pytest
pytest tests/test_gamelog_parser.py -v
```

## Architecture

```
┌─────────────────────────────────────┐
│ DWARF FORTRESS (DFHack Lua)         │
│  storyteller-begin.lua              │  ← First-time setup + snapshot
│  storyteller-events.lua             │  ← Continuous event monitoring
│  storyteller-snapshot.lua           │  ← Delegates to begin --snapshot-only
│    ↓                                │
│  JSON files → storyteller_events/   │  ← Per-world subfolders
│               {world_folder}/       │
└─────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────┐
│ PYTHON BACKEND                       │
│  context/loader.py                  │  ← Loads snapshots + events + legends
│    ↓                                │     Merges sibling folders (region2 + autosave 1)
│  context/narrative_formatter.py     │  ← Interprets raw data → prose descriptions
│  context/notes_store.py             │  ← Player notes (suspicion, fact, rumor, etc.)
│    ↓                                │
│  stories/chronicle.py               │  ← Season chronicles with event context
│  stories/biography.py               │  ← Dated bio entries that track change
│  stories/saga.py                    │  ← World history from legends
│    ↓                                │
│  llm/ (Claude/OpenAI/Ollama)        │  ← LLM-agnostic provider layer
│    ↓                                │
│  web/app.py (FastAPI)               │  ← Web UI server
│  web/templates/ (Jinja2)            │  ← Fantasy parchment theme
└─────────────────────────────────────┘
```

**Dependency direction**: `ingestion → schema ← context → llm → stories ← web`. Never import from `llm/` in `ingestion/` or `schema/`.

## Portrait System

Generates pixel-accurate dwarf portraits from DF's own sprite sheets. Self-contained module at `src/df_storyteller/portraits/` — no dependencies on the rest of df_storyteller. Also published as standalone package [df-portrait-compositor](https://github.com/Been012/df-portrait-compositor).

- **5 races**: DWARF, ELF, HUMAN, GOBLIN, KOBOLD — each with race-specific sprite sheets and palettes.
- **Compositor** (`compositor.py`): Parses DF's graphics definition file, evaluates layer conditions (tissue color/length/shaping, body part modifiers, equipment, syndromes), composites matching tiles with palette recoloring.
- **Age groups**: BABY (<1), CHILD (1-12), PORTRAIT (12+) layer sets with different sprite sheets.
- **Clothing**: Dye color detection via `dye_profile.color_index`, HSV tinting at 40% saturation, quality tier tiles, material flag matching.
- **Creature sprites** (`creature_sprites.py`): 504 pre-made creature portraits (domestic, surface, underground, aquatic).
- **Key gotchas**: Male tissue layout [6,7]=HAIR not [3]=MOUSTACHE. Hair palette source row varies per race (human=row 12, others=row 0). DF graphics uses `PONY_TAILS` (plural) but tissue enum uses `PONY_TAIL`. `BP_MISSING` must be rejected or scar tiles render as "hats".
- **Unsolved**: Procedural creature portraits (demons, forgotten beasts) — `pcg_layering` enum not mapped by DFHack. Code saved in `experimental/beast_compositor.py`.

## Key Architecture Decisions

### DFHack Data Bridge
- **File-based communication** between DF and Python. Lua writes JSON files, Python reads them. No direct RPC — simpler and more reliable.
- **Per-world subfolders** under `storyteller_events/`. The folder name comes from `dfhack.world.ReadWorldFolder()` which can change between loads (e.g. `region2` vs `autosave 1`). The Python loader merges folders with matching fortress identity (same `civ_id` + `fortress_name`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Been012/df-storyteller](https://github.com/Been012/df-storyteller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
