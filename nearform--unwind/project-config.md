---
trigger: always_on
description: Guidance for working in this repo.
---

# CLAUDE.md

Guidance for working in this repo.

## What Unwind is

A Claude Code plugin that **reverse-engineers a codebase so an AI agent can rebuild
it in a different stack**, preserving API contracts, business logic, and the data
model. Its differentiator: every documented item is tagged `[MUST]` / `[SHOULD]` /
`[DON'T]` for rebuild prioritization.

Unwind is **hybrid**: deterministic scripts (`@unwind/core`, tree-sitter) own the
verifiable facts (file inventory, structural symbols, contracts, import graph,
layer assignment); LLM sub-agents add the semantic rebuild docs. Completeness is
then **verified by set arithmetic** (`scan − docs`), not asserted. Every script
**degrades gracefully** — if Node/pnpm/core is unavailable, skills fall back to the
legacy pure-LLM flow and say so.

## Repo layout

```
.claude-plugin/         plugin.json + marketplace.json (manifests stay at root)
packages/
  core/                 @unwind/core — the deterministic engine (TypeScript → dist/)
    src/
      scan/             enumerate (git ls-files + walk), language/category tables, repo-info
      structure/        tree-sitter plugin + per-language extractors (ts/js, python, rust, java, c#)
      imports/          internal import-map resolution
      layers/           rebuild-layer-map (file → layer) + contract-detectors (tables/endpoints)
      fingerprint/      structural + content fingerprints (incremental updates)
      manifest/         scan-manifest schema, build-manifest, candidates (shared id scheme)
      graph/            coverage diff, rebuild-graph schema + build-graph,
                        rebuild-state schema + rebuild-verification (source→target diff)
      index.ts          public barrel (export blocks grouped by increment)
  dashboard/            @unwind/dashboard — React + React Flow + ELK (Vite), consumes rebuild-graph.json
skills/
  scripts/              bundled .mjs the skills invoke (scan, seed-layers, verify-coverage,
                        build-graph, detect-changes, merge-rebuild-map, verify-rebuild,
                        deploy-gh-pages)
                        + _core.mjs + _resolve-plugin-root.sh
  *                     markdown skills (uw-start entry point, uw-scan, uw-analyze,
                        uw-analyze-* layer specialists, uw-verify, uw-complete,
                        uw-plan, uw-graph, uw-dashboard, uw-publish, uw-build, uw-build-layer,
                        uw-refresh, uw-help, analysis-principles, rebuild-principles)
```

`dist/` and `*.tsbuildinfo` are gitignored — the core is **lazily built on first
use** (`ensure_unwind_core`). `pnpm-lock.yaml` IS committed. We standardize on pnpm.

## The pipeline (see README + docs/pipeline.svg)

`scan.mjs` → **scan-manifest.json** (ground truth) → `seed-layers.mjs` → per-layer
candidate checklists → LLM layer specialists write tagged docs with anchor-id
headings → `verify-coverage.mjs` does the deterministic `manifest − docs` diff →
`gaps.md` → `uw-complete` fills them (loop to 100%) →
`plan-brief.mjs` → **plan-brief.json** (the deterministic facts) → `uw-plan`
**interviews the user** (grilling-style: target stack, re-use, phasing, risk) →
REBUILD-PLAN.md (+ `rebuild-decisions.json`) → `build-graph.mjs` →
**rebuild-graph.json** → `uw-dashboard`. `detect-changes.mjs` (fingerprints) drives
incremental refresh. To **share** the dashboard, `uw-publish` (optional) builds it at
the project's GitHub Pages sub-path and commits it into an `unwind/` subdir of the
project's `gh-pages` branch (`deploy-gh-pages.mjs`) — never blatting existing branch
content; viewable at `https://<owner>.github.io/<repo>/unwind/`.

Then **execution** (optional): `uw-build` interviews the user (scope/order/target),
dispatches technology-agnostic `uw-build-layer` subagents that reproduce each slice's
`[MUST]` contracts in the target stack and write per-slice source→target maps →
`merge-rebuild-map.mjs` folds them into **rebuild-state.json** (+ derives
`rebuild-progress.json`) → `verify-rebuild.mjs` re-scans the **target** repo and
diffs it against the source graph → **rebuild-verification-graph.json** + `rebuild-gaps.md`
(completeness over `[MUST]`). Builds run step-through or `/loop /uw-build`
(loop-until-verified; the deterministic completeness % is the termination signal).

Artifacts live under the **source** repo's `docs/unwind/`:
`architecture.md`, `layers/**`, `REBUILD-PLAN.md`, `rebuild-graph.json`,
`rebuild-verification-graph.json`, `rebuild-gaps.md`, and
`.cache/` (`scan-manifest.json`, `meta.json`, `changes.json`, `seeds/`, `coverage/`,
`plan-brief.json`, `rebuild-decisions.json`, `rebuild-state.json`, `rebuild-map/`,
`rebuild-progress.json`, `target-scan/`). Rebuilt code lives in the separate **target** repo.

## Commands

```bash
pnpm install                                   # workspace deps (incl. tree-sitter grammars)
pnpm --filter @unwind/core build               # or: cd packages/core && tsc -p tsconfig.json
pnpm --filter @unwind/dashboard build          # vite production build
UNWIND_GRAPH_DIR=<project> pnpm --filter @unwind/dashboard dev   # dashboard on 127.0.0.1:5174

node skills/scripts/scan.mjs <projectRoot>            # → docs/unwind/.cache/scan-manifest.json (+ meta.json)
node skills/scripts/seed-layers.mjs <projectRoot>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nearform/unwind](https://github.com/nearform/unwind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
