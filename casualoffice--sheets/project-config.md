---
trigger: always_on
description: A web-based **Excel-equivalent** with real-time collaborative editing, built on **Univer OSS** (Apache-2.0). The goal UX is Microsoft Excel / Office — ribbon, formula bar, file-centric flow — not Google Sheets.
---

# CLAUDE.md — instructions for Claude Code in this repo

## What this project is

A web-based **Excel-equivalent** with real-time collaborative editing, built on **Univer OSS** (Apache-2.0). The goal UX is Microsoft Excel / Office — ribbon, formula bar, file-centric flow — not Google Sheets.

## What's in scope

- Upload `.xlsx` → open in browser session → multi-user co-edit → download `.xlsx`.
- In-memory sessions only. No database. No accounts.
- Office-style UI shell built on top of Univer's grid + formula engine.

## What's out of scope (do not propose, do not build)

> **Note**: parts of this section are out of date — Phase C personal mode, Phase D WOPI, autosave, and version history have all shipped since this list was written. The remaining "do not build" entries below are still binding.

- **AI / LLM features** — the user will plug in a self-hosted LLM later through Univer's command bus. Don't pre-design for it.
- **Mobile** — supported as a **viewer + light editor** down to ~360 px (iPhone SE+). Open files, scroll, single-cell value edits, basic formatting via the menu strip + compact toolbar, sheet switching. NOT supported: chart insert dialogs, pivot field-list, complex formula composition, or any flow that needs hover + right-click on phone. Breakpoints live in `apps/web/src/styles.css` at `@media (max-width: 720px)` and `@media (max-width: 480px)`. iOS Safari requires input font-size ≥ 16 px to avoid focus-zoom; honour that on any input inside the chrome. Univer's canvas owns its own touch gestures — don't try to wrap them.
- **Univer Pro features** — collab, xlsx I/O, charts, pivots, print, history are all paid in Univer's commercial offering. We are *not* using Univer Pro. We build the collab + xlsx layers ourselves on OSS.

## Required reading before substantive work

1. [`PLAN.md`](./PLAN.md) — phased plan and estimates.
2. [`docs/ARCHITECTURE.md`](./docs/ARCHITECTURE.md) — how the pieces fit today.
3. [`docs/SDK_ARCHITECTURE.md`](./docs/SDK_ARCHITECTURE.md) — **target** architecture: this repo's primary purpose is an embeddable editor SDK other engines attach to (the Excalidraw model: package *is* the editor, opt-in collab server, thin localStorage host).
4. [`docs/SDK_MIGRATION_PIPELINE.md`](./docs/SDK_MIGRATION_PIPELINE.md) — phased path to that target (Phase 0 = Univer 0.25).
5. [`docs/RESEARCH.md`](./docs/RESEARCH.md) — Univer technical brief with file path references.
6. [`SKILLS.md`](./SKILLS.md) — build/test/verify/release/fork workflows. [`CONTRIBUTING.md`](./CONTRIBUTING.md) — contribution + verification gate.
7. [`docs/RELEASING.md`](./docs/RELEASING.md) — the **two independent release lines**: Docker app (`casualoffice/sheets`, `vX.Y.Z` tags, latest 0.3.2) vs npm SDK (`@casualoffice/sheets`, Changesets, latest 0.8.0). They version separately — don't conflate them.

## Hard rules

### `vendor/univer-revamp/` is our fork — modifiable AND wired into the build

- It is a git submodule of [`CasualOffice/univer-revamp`](https://github.com/CasualOffice/univer-revamp), our long-term fork of `dream-num/univer` (currently v0.25.0, branch `casual-sheets/0.25`).
- **You may modify files under `vendor/univer-revamp/`** — commits land in the fork.
- **It IS wired into the build.** The root `package.json` `pnpm.overrides` block `link:`s every `@univerjs/*` package to `vendor/univer-revamp/packages/*`, so the app + SDK resolve Univer to the fork, not npm. Edits here are real local runtime changes — rebuild the fork (`pnpm fork:setup`, which builds + swaps the package.jsons to `lib/`) for them to take effect. (`vendor/univer.stale/` is an old pre-submodule clone — ignore it.)
- Read it freely. Cite file paths and line numbers when explaining Univer internals; `vendor/univer-revamp/packages/.../file.ts:LINE` format applies.
- Active perf plan lives at [`docs/UNIVER_FORK_PERF.md`](./docs/UNIVER_FORK_PERF.md).

### Pin Univer version

- Per the research brief, Univer's `IWorkbookData` shape and plugin contracts change across minor versions, with strict version validation between plugins.
- Pin **all** `@univerjs/*` packages to the exact same version. Never mix. Current pin: **0.25.0** (`apps/web/package.json` + `packages/sdk/package.json`).
- **0.25 upgrade (Phase 0): ✅ done.** The vendored submodule `vendor/univer-revamp` (remote `CasualOffice/univer-revamp`) sits on branch `casual-sheets/0.25`, at the `v0.25.0` release with **six** custom commits on top — 2 feature (paste-merge preservation, filtered-dropdown visibility) + 4 perf (font-cache LRU, merge-range row-bucket index, header hit-test index, setStylesCache span). Every `@univerjs/*` pin is `0.25.0` and the `pnpm.overrides` block links to the fork packages. When upgrading again (e.g. 0.26): branch `casual-sheets/0.26` from the upstream tag, cherry-pick the six commits, bump every pin, re-audit `pnpm.overrides`, retest. See `docs/SDK_MIGRATION_PIPELINE.md` Phase 0.

### Use the collab hook Univer designed for it

- Do **not** hook into UI events to capture changes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CasualOffice/sheets](https://github.com/CasualOffice/sheets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
