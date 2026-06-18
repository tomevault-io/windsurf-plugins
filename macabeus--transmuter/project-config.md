---
trigger: always_on
description: Transmuter — a TypeScript library + CLI that permutes C / C++ / Pascal source code to match target assembly, and refines already-matching code to remove code smells.
---

# CLAUDE.md

Transmuter — a TypeScript library + CLI that permutes C / C++ / Pascal source code to match target assembly, and refines already-matching code to remove code smells.

## Commands

All commands run from the repo root. **Package manager is pnpm** (workspace protocol). Never use `npm` or `yarn`.

```bash
pnpm install                               # install all workspaces
pnpm run build                             # build all packages (pnpm -r run build)
pnpm run check-types                       # tsc --noEmit for core + cli
pnpm run lint                              # eslint packages/
pnpm run format                            # prettier write
pnpm test                                  # vitest run (core + cli projects)
pnpm --filter @transmuter/core run test:watch
pnpm run test:fixture                      # run all 7 real-compiler fixtures
pnpm run test:fixture -- --fade-out-controller   # single fixture

# Core package dev (watch build, no DTS)
pnpm --filter @transmuter/core run build:esm
pnpm --filter @transmuter/core run dev     # bun --watch run build.ts

# CLI — dev mode auto-builds core first (predev hook)
pnpm --filter @transmuter/cli run dev -- match path/to/source.c

# Webapp
pnpm run build:webapp                      # single-file HTML bundle
pnpm run dev:webapp -- ./session-report.json   # live dev with a report JSON

# Compiler toolchains (one-time, required for fixtures + real runs)
./setup-compilers.sh                       # builds agbcc + IDO 7.1 submodules
```

## Package map

| Package | Path | What it is | Entry point |
|---|---|---|---|
| `@transmuter/core` | `packages/core/` | Search engine, rules, guidelines, scoring, session store. Published library. | `src/index.ts` |
| `@transmuter/cli` | `packages/cli/` | `transmuter` binary — Ink TUI + Hono HTTP control server. | `src/index.ts` |
| `@transmuter/webapp` | `packages/webapp/` | Static React single-file bundle for viewing `SessionReport`/`RefinementReport` JSON. Private. | `src/main.tsx` (via `vite.config.ts`) |

CLI commands: `match`, `refine`, `profile-detect`, `ctl`. There is no standalone `reduce` or `rules` subcommand — the reducer runs as the pre-step of `match` (skip with `--no-reduce`), and rules are listed via `profile-detect`.

## Architecture summary

- **Core pipeline**: `MutationSearch` runs N concurrent slots. Each slot loop = `pool.select() → engine.mutate() → dedup → compile → scorer.scoreWithAssembly() → pool.report() → emit events`. Bottleneck is compilation (subprocesses via `Compiler` class).
- **Candidate graph**: `Pool` manages a tree of immutable `CandidateNode`s connected by `parentId`. On score improvement, the pool **forks** — creates a new candidate + new `MutationTarget`. The parent target keeps exploring. Fork dedup tuple: `(scoreDelta, ruleId, line, column)`. Genesis never mutates in place.
- **Rules vs guidelines**: Rules (49 built-in) are mutation plugins selected by weighted Thompson Sampling filtered by diff-type affinity. Guidelines (4 built-in) detect a `Violation`, know how to strip it, and drive `Refiner` sub-sessions that re-match while preventing re-introduction.
- **Scoring**: `Scorer` wraps `objdiff-wasm`. `scoreWithAssembly()` returns `{ score, breakdown: DiffBreakdown, assembly, assemblyDiff }` in one pass. `DiffBreakdown` = insert + delete + replace + opMismatch + argMismatch.
- **Session report**: `SessionStore` captures `MutationSearchEvent`s, produces a `SessionReport` JSON consumed by the webapp. `RefinementStore` plays the same role for `transmuter refine`.
- **HTTP API**: `--api` starts a Hono server on localhost with common read/control endpoints + mode-specific extras. Writes a `transmuter-control.json` discovery file. `transmuter ctl` is the client.

For the full design, read `.claude/docs/architecture.md`.

## Key conventions

- **pnpm workspaces.** `workspace:*` for internal deps. Lockfile is `pnpm-lock.yaml`.
- **Path alias `~` → `./src`** inside each package (not a global alias — each package's `tsconfig.json` declares its own). The webapp uses `@core` → `../core/src` to import core types/helpers directly from source (no built package needed for the webapp to compile).
- **ESM only.** `"type": "module"` everywhere. Imports must end in `.js` even for TS files (`import x from './foo.js';`).
- **Strict TS.** `strict`, `strictNullChecks`, `noUncheckedIndexedAccess`. Writing array/map access without a null check is a type error.
- **Test files**: `*.spec.ts` co-located with the source file. Shared fixtures live in `test-utils.ts` next to the specs (not under `__fixtures__/`).
- **No mocks.** Scoring/compiler tests shell out to the real `arm-none-eabi-as`, run real `objdiff-wasm`, and build real ELF objects. Follow this for any new scoring/compiler test.
- **Rule/guideline plugins** are plain exported objects implementing `Rule` / `Guideline`. They declare `languages: readonly Language[]` (`'c' | 'cpp' | 'pascal'`) — the engine filters by language automatically. Register in `packages/core/src/rules/built-in/index.ts` or `packages/core/src/guidelines/built-in/index.ts`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [macabeus/transmuter](https://github.com/macabeus/transmuter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
