---
trigger: always_on
description: Context for AI coding agents, tool-agnostic (Claude Code loads it via the
---

# AGENTS.md — Overseer

Context for AI coding agents, tool-agnostic (Claude Code loads it via the
`@AGENTS.md` import in the CLAUDE.md stub). Keep short and current.
**Binding conventions & gotchas: [docs/ai/rules.md](docs/ai/rules.md) — code and
comments are written in ENGLISH** (German only in translation data, test
fixtures, and user-facing UI copy).

**Package guides live in [docs/ai/](docs/ai/)** (moved out of the
source tree so the shipped code stays clean — nothing under `src/` but code).
READ the matching guide BEFORE working in that package:
[src.md](docs/ai/src.md) (plugin entry) ·
[overseer.md](docs/ai/overseer.md) (package root, bridge, config) ·
[cinema.md](docs/ai/cinema.md) (c4d host glue, every module + gotchas) ·
[core.md](docs/ai/core.md) · [naming.md](docs/ai/naming.md).

## What this is

Cinema 4D plugin (tested with 2023 & 2024) for any kind of project: **analyzes** the scene for
insights, **normalizes object names**, and manages layers, tags, assets,
materials & textures. One UI: a web frontend (Vite/React) served by the plugin.

Tested against the user's ~2.3 GB production scene.

**Key decision: plugin, NOT headless.** `c4dpy.exe` hangs on a license prompt →
all code runs as a plugin inside the licensed C4D GUI (details in rules.md).

## Architecture

**Core principle: pure domain logic strictly separated from `c4d`.**
`src/overseer/` never imports `c4d` → testable in CI. Only these modules import
`c4d` (never loaded by tests): `cinema/` (adapter/webapi), `bridge/`.

```
src/
  overseer.pyp     Loader. Registers ONE command "Overseer" that
                          starts the server + opens the web UI (the only UI).
  overseer/
    config.py             config.json schema 3 (migrate_config reads v1/v2 forever;
                          per-section "accepted as-is" keeps map)
    bridge/               [c4d] HTTP server (BG thread) + main-thread queue + progress
                          state; one file per class (progress/mainthread/reload/
                          server/dialog). PROCESS SINGLETON — the whole package
                          is excluded from hot-reload.
    core/
      model.py            SceneNode / SceneTree (pure hierarchy)
      ops.py              plan_renames()/plan_layers() + RenameOp/ReparentOp/LayerOp
      keeps.py            per-section "accepted as-is" lists (filter_kept/set_section_keeps)
      analyzer.py         SceneTree -> SceneReport (single pass)
    naming/
      casing.py           Tokenizer, casing detection, language heuristic (was naming.py)
      convention.py       NamingConvention (casing/language/numbering), disambiguate()
      translations.py     DE<->EN dictionary + add_translations()
      translate.py        Language-only rename proposals
      detect.py           Auto-detect existing scheme (style/language/pad + confidence)
    cinema/               [c4d] host glue
      adapter/            doc <-> SceneTree; rename/reparent/plan/layers with undo.
                          One file per domain class (readers/scene/materials/
                          previews/texpaths/texresize/layers/apply/journal)
      webapi.py           JSON API; hot-reloaded per request. Scene-tree +
                          preview caches live on the `overseer` package
                          (survive hot-reload; invalidated by the doc dirty
                          counter, cleared by POST /api/reload). Every slow op
                          publishes a progress label (_OP_LABELS -> /api/progress).
  web/                    Vite build output (gitignored; deployed by deploy.ps1)
frontend/                 Vite/React/TypeScript source (App.tsx, tabs/, components/, hooks/useOrganizer.ts)
  STYLEGUIDE.md           UI vocabulary: every reusable block, its class + markup
                          (.section-head, sidebar text ranks, buttons, colour meaning).
                          READ BEFORE touching CSS — reuse a block, never fork a near-copy.
tests/                    pytest, runs WITHOUT c4d
.github/workflows/ci.yml  4 jobs: plugin-lint (ruff), plugin-test (pytest, Python 3.11 =
                          C4D 2024 runtime; ruff enforces 3.9 syntax = C4D 2023),
                          frontend-lint (tsc), frontend-test (vitest + vite build);
                          runs on main + PRs
.github/workflows/release.yml  main = RELEASE BRANCH: every main push gates, builds
                          Overseer-<version>.zip and replaces the release of the
                          version stamped in the repo (tag moves along; version
                          gate checks pyproject/__init__/package.json agree).
                          main is PROTECTED (PR + green CI required, no direct
                          push): ALL work happens on feature/<topic> branches
                          (no permanent dev branch), a PR into main publishes.
                          The repo is lukasguziel/overseer (public home of
                          code, issues and releases — no mirror anymore).
.claude/skills/deploy/    deploy skill incl. deploy.ps1 (copies .pyp + overseer/ +
                          web/ to the plugin dir) + machine-local
                          deploy.config.json (gitignored)
```

## Plugin IDs / port (overseer.pyp)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lukasguziel/Overseer](https://github.com/lukasguziel/Overseer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
