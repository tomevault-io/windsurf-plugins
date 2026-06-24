---
trigger: always_on
description: **claude-mem-sync** is a TypeScript CLI tool and Claude Code plugin that enables curated, filtered, scored team memory sharing for [claude-mem](https://docs.claude-mem.ai). It syncs AI memories across developers via git.
---

# CLAUDE.md — claude-mem-sync

## What this project is

**claude-mem-sync** is a TypeScript CLI tool and Claude Code plugin that enables curated, filtered, scored team memory sharing for [claude-mem](https://docs.claude-mem.ai). It syncs AI memories across developers via git.

- **Package name**: `claude-mem-sync`
- **CLI binary**: `mem-sync`
- **License**: MIT
- **Runtime**: Bun (v1.0+) or Node.js (v18+)
- **SQLite**: `bun:sqlite` on Bun, `better-sqlite3` on Node.js (auto-detected via `src/core/compat.ts`)
- **Runtime deps**: Zod (config validation), better-sqlite3 (optional — only needed on Node.js)

## Architecture

```
Developer Machine                    Shared Git Repo
┌─────────────────────┐              ┌──────────────────────┐
│ claude-mem SQLite DB │◄─ read-only─│ Export Pipeline       │
│ (~/.claude-mem/)     │             │ (filter + score)     │──► contributions/
│                      │             └──────────────────────┘
│ PostToolUse Hook ────►  access.db  │ Import Pipeline      │◄── merged/latest.json
│ (tracks real usage)  │             │ (dedup + integrity)  │
└─────────────────────┘              └──────────────────────┘
                                     GitHub Action merges contributions → merged/
                                                                          │
                                     ┌──────────────────────┐             │
                                     │ Profile Pipeline     │◄────────────┤
                                     │ (per-dev metrics)    │──► profiles/│
                                     └──────────────────────┘             │
                                     ┌──────────────────────┐             │
                                     │ Distillation Pipeline│◄────────────┘
                                     │ (LLM → rules + KB)  │──► distilled/
                                     └──────────────────────┘
```

**Key invariants**:
- Export pipeline and hook are **read-only** on claude-mem's DB
- **Only the import pipeline writes** to claude-mem's DB (with transaction rollback safety)
- access.db (`~/.claude-mem-sync/access.db`) is our own tracking DB — never touches claude-mem's schema
- All SQLite connections use `PRAGMA busy_timeout = 5000` for WAL contention handling
- Dedup uses composite key: `sdk_session_id + title + created_at_epoch` (NOT the auto-increment `id`)

## Project structure

```
src/
  cli.ts                  — Entry point: command router (parseFlags + dispatch)
  types/
    config.ts             — Zod schemas + TS types for config (incl. profiles + distillation)
    observation.ts        — Observation, ScoredObservation, ExportFile interfaces
    merge-state.ts        — .merge-state.json types
    profile.ts            — Developer profile, team overview, concept map types
    distillation.ts       — Distilled rules, knowledge sections, report, feedback types
  core/
    constants.ts          — Paths, defaults, type weights, dedup key fields
    logger.ts             — Debug/error → file, info/warn → console
    config.ts             — Load, validate, resolve config from ~/.claude-mem-sync/config.json
    mem-db.ts             — Read-only + writable access to claude-mem's SQLite DB
    access-db.ts          — Our access_log, import_log, export_log DB
    filter.ts             — OR-based filter: type/keyword/tag matching
    scoring.ts            — Eviction scoring (hook mode + passive mode)
    merger.ts             — Dedup + cap enforcement with eviction
    git.ts                — Git operations via Bun.spawn (array args, NO shell)
    scheduler.ts          — Cross-platform cron/launchd/schtasks generation
    profiler.ts           — Profile computation from contribution/merged files
    distiller.ts          — LLM API integration + output rendering
    dashboard-server.ts   — HTTP server with 16 API routes + 9-tab SPA
    dashboard-profiles.ts — Profile/team API handlers
    dashboard-distilled.ts — Distillation rules/KB/feedback API handlers
    analytics.ts          — Analytics computations for dashboard
    prompts/
      distillation-system.ts — System + user prompt templates for distillation
  commands/
    init.ts               — Interactive setup wizard
    config.ts             — Show current configuration
    setup-repo.ts         — Scaffold a shared team memory repository
    add-project.ts        — Add a new project to existing config
    update-project.ts     — Update an existing project's config
    export.ts             — Export filtered memories to git
    import.ts             — Import merged memories into local DB
    preview.ts            — Dry-run export preview
    maintain.ts           — DB maintenance (backup, prune, vacuum, integrity)
    status.ts             — Health check dashboard
    schedule-install.ts   — Install OS scheduled tasks
    schedule-remove.ts    — Remove OS scheduled tasks
    ci-merge.ts           — CI-only merge command for GitHub Action
    profile.ts            — Generate developer knowledge profiles
    distill.ts            — LLM-powered knowledge distillation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lopadova/claude-mem-sync](https://github.com/lopadova/claude-mem-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
