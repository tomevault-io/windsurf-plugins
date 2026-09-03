---
trigger: always_on
description: Guidance for AI coding assistants (Cursor, Claude Code, Aider, etc.) working
---

# AGENTS.md

Guidance for AI coding assistants (Cursor, Claude Code, Aider, etc.) working
on the Unlock All Classes mod. If you're a human, read the [README](README.md)
first -- this file assumes you've already seen the basics.

> For Claude Code specifically, [CLAUDE.md](CLAUDE.md) points at this file.
> Everything here applies to every agent.

---

## Purpose

Allow any playable race to create and play any playable class, with a design that is
**maintainable, revertable, and free of mystery binaries**, and that is **reusable by other
server operators** running stock or lightly-customized AzerothCore installations.

### Goals
- No unexplained binary DBC files. Every artifact is either generated from a known source or is
  human-readable SQL.
- No hand-applied, irreversible SQL. Everything is applied by AzerothCore's DB updater and has a
  companion revert.
- **Zero server-side binary DBC edits.** (See [engineering doc §3](docs/mod-uac-engineering-implementation.md#3-source-grounded-findings).)
- Exactly three small, universal client artifacts (generated MPQs under `client-patch/`), producible in pure Python.
- Deterministic and reproducible: generation happens against a pinned canonical source, or against
  the operator's own installed DBCs.

### Non-goals
- Custom NPC dialogue beyond the generated trainers and quest patches (unrelated to this module).

**Resolved / in scope (formerly deferred):** `mod-playerbots` already spawns new combos once
mod-uac's `playercreateinfo` rows are applied. Starter-zone class trainers ship in
`mod_uac_starter_trainers.sql` (see [docs/trainer_worksheet.md](docs/trainer_worksheet.md)).

---

## Where things live

```
mod-uac/
  data/sql/db-world/            # install SQL (auto-applied by AC updater)
  data/sql/db-uninstall/        # companion revert SQL (manual; dirname must not contain "world")
  tools/aracgen/                # generator package
      dbc.py  sources.py  matrix.py  kits.py
      emit_skill.py  emit_player.py  emit_totem.py  emit_class_quest.py
      emit_totem_quest.py  emit_hunter_pet.py  emit_trainers.py  emit_client.py  mpq.py
      snapshot.py  trainer_catalog.py  capital_trainer_catalog.py  schema_emit.py
      class_quest_catalog.py  totem_quest_catalog.py
  tools/capture_snapshot.py     # world DB snapshot capture (PyMySQL)
  tools/generate_local.py       # LocalDbcSource     -> operator SQL + standard MPQ
  tools/generate_canonical.py   # CanonicalDbcSource(v19) -> checked-in SQL + client MPQs
  data/snapshot/                # baked world snapshot (schemas + trainer extracts)
  data/item_prototypes.json     # minimal outfit item class/subclass lookup (from snapshot refresh)
  data/trainer_overrides.yaml   # optional trainer placement overrides (starter + capital)
  tools/requirements.txt
  client-patch/unlock-only/patch-z.mpq   # CharBaseInfo + SkillRaceClassInfo (equip + UI normalization)
  client-patch/standard/patch-z.mpq      # above + v19 CharStartOutfit + overlays
  client-patch/enhanced/patch-z.mpq      # above + HD baseline CharStartOutfit + overlays
  data/client/hd_outfit_templates.json   # deduplicated HD stock outfit templates (54)
  data/client/hd_outfit_stock_index.json # 126 stock rows -> template_id
  CMakeLists.txt                # data-only module stub
  README.md
```

---

## Coding conventions

- **Python 3.11**. Use modern syntax: `X | Y` unions, `list[int]`, `datetime.UTC`
  instead of `datetime.timezone.utc`. Ruff will tell you.
- **Comments explain why, not what.** A comment like `# increment counter` on
  `counter += 1` is noise. A comment explaining a non-obvious trade-off is
  gold.
- **Write the test before you believe the fix.** Passing tests will catch
  more regressions than any amount of local eyeballing. Run from `tools/`:
  `python -m pytest` and `python -m ruff check .`.
- **Emitters produce install + uninstall pairs.** SQL under
  `data/sql/db-world/` and `data/sql/db-uninstall/`; never hand-edit
  checked-in SQL — regenerate via `generate_canonical.py` (or
  `generate_local.py` for operator baselines).
- **Schema-driven SQL.** World-table emitters (`emit_skill`, `emit_player`,
  `emit_totem`, `emit_class_quest`, `emit_hunter_pet`, `emit_trainers`) render
  INSERT/REPLACE/UPDATE rows from the baked snapshot's `TableSchema` via
  `schema_emit.py` — column lists and defaults come from the operator's world DB
  capture (or AC base DDL bootstrap), not hardcoded in emitters.
- **Source world facts from the snapshot, don't hardcode them.** Anything the
  emitter needs to *know about the game world* — which NPCs exist and where they
  spawn, what a trainer teaches, a quest's stock `AllowableRaces`/level, item
  categories, coverage of a zone — must come from the captured snapshot
  (`tools/capture_snapshot.py` → `data/snapshot/`), the same way starter and
  capital trainers are placed by *reading* captured trainers rather than a table
  of entries/coordinates. If you find yourself writing a literal NPC entry,
  spawn coordinate, or "this city already has class X" list into an emitter or
  catalog, stop: capture it instead (extend `capture_trainer_data` /
  `SCHEMA_TABLES` and refresh the snapshot). The **only** things that may be

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [berubejd/mod-uac](https://github.com/berubejd/mod-uac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
