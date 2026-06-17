---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Soulmask Codex — a full-stack game-data reference site for the Soulmask survival game. Three layers:

1. **Pipeline** (Python 3, no deps) — reverse-engineers UE4 modkit `.uasset` files into `Game/Parsed/*.json`
2. **Backend** (Go + chi) — serves a JSON API from a SQLite DB at `backend/internal/api/`
3. **Frontend** (React + Vite + TypeScript) — SPA in `web/`, embedded into the Go binary for prod

Deployed as a single binary on Fly.io (`soulmask-codex`, region `ams`). Live at **soulmask-codex.com**.

Docs: `docs/DATA.md` (data shapes, fill rates, cross-ref maps), `docs/DESIGN.md` (game-concept glossary).

## Key paths

| What              | Path                         | Notes                                    |
| ----------------- | ---------------------------- | ---------------------------------------- |
| SQLite DB         | `data/app.db`                | **the only DB file — do not create others** |
| Translations      | `data/translations/*.json`   | manual Chinese→English overrides         |
| Parsed JSON       | `Game/Parsed/*.json`         | committed pipeline output                |
| Exported tables   | `Game/Exports/*.json`        | committed UE4 DataTable exports          |
| Raw BP exports    | `uasset_export/`             | gitignored (~800 MB)                     |
| Backend source    | `backend/`                   | Go module `github.com/rubensayshi/soulmask-codex` |
| Frontend source   | `web/`                       | React + Vite, pnpm                       |
| DB schema         | `backend/internal/db/schema.sql` | single source of truth for tables    |
| Generated DB code | `backend/internal/db/gen/`   | sqlc output, do not edit                 |

**Do not create `.db` files anywhere else** (e.g. `data/soulmask.db`, `soulmask.db`). The backend flag defaults to `--db ../data/app.db`; `build_db.py` writes to `data/app.db`. Nothing else.

## Pipeline (two-stage, two-platform)

```
Modkit .uasset files  ──►  [Windows-only export]  ──►  uasset_export/  &  Game/Exports/
                                                       │
                                                       ▼
                                               [any platform parsing]
                                                       │
                                                       ▼
                                               Game/Parsed/*.json  ──►  data/app.db
```

**Stage 1 (Windows only, requires modkit at `C:\Program Files\Epic Games\SoulMaskModkit`):**
- `pipeline/run_export.bat` → runs `pipeline/export_tables.py` inside `UE4Editor-Cmd.exe` to export 11 DataTables to `Game/Exports/*.json`.
- UAssetGUI (manual, GUI tool) → exports BP_PeiFang / BP_DaoJu / BP_KJS `.uasset` files to `uasset_export/**/*.json.gz` (gitignored, ~800MB).

**Stage 2 (host Python 3, no deps):**
```bash
python3 pipeline/parse_exports.py     # drops.json     (from Game/Exports/)
python3 pipeline/parse_recipes.py     # recipes.json   (from uasset_export/Blueprints/PeiFang/)
python3 pipeline/parse_items.py       # items.json     (from uasset_export/Blueprints/DaoJu/)
python3 pipeline/parse_tech_tree.py   # tech_tree.json (from uasset_export/Blueprints/KeJiShu/)
```

Parsers are independent — run any one in isolation. Outputs are committed to git (`Game/Parsed/`).

**Stage 3: `pipeline/build_db.py`** — reads `Game/Parsed/*.json` + `data/translations/*.json`, writes `data/app.db` using `schema.sql` as the DDL source. Idempotent (drops and recreates).

**After running any individual parser, always run `make db`** to ensure downstream enrichment steps (classification, food buffs) and the SQLite rebuild are not lost. Running `parse_items.py` alone strips the `buffs` field that `parse_food_buffs.py` adds.

## Two distinct parsing strategies

The code uses **two different approaches** depending on whether the data was exported via UE4Editor or UAssetGUI — don't mix them up:

1. **DataTable rows (`parse_exports.py`)** — regex over UE4 property-export **text** (strings like `((SelectedRandomProbability=30, BaoNeiDaoJuInfos=((DaoJuClass=...)))`). See `split_top_level_parens` / `parse_daoju_bag_content`. Item refs get resolved via `parse_localization.load_names()` (PO-file lookup, keyed on normalized asset paths).

2. **Blueprint assets (`parse_recipes.py`, `parse_items.py`, `parse_tech_tree.py`)** — walk UAssetAPI's tagged-property **JSON tree**. Item references are negative ints into the `Imports` table; resolve with the shared helper:

```python
def resolve_import_path(imports, ref):
    # negative ref → index into Imports; OuterIndex chains to the Package import
    # that holds the /Game/... path
```

This helper (and `find_props` / `get_prop` / `text_zh`) is copy-pasted into each BP parser — they're ~identical but not factored out. Don't refactor casually; the duplication is intentional to keep each parser independently runnable.

## Non-obvious things


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rubensayshi/soulmask-codex](https://github.com/rubensayshi/soulmask-codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
