---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

PerfectFit is a local-first, guided wizard for calibrating filament profiles for Orca
Slicer and Bambu Studio. It's a TypeScript/Vite static web app with no backend/accounts/
analytics — all data lives in the browser (IndexedDB + localStorage) — optionally packaged
as a native desktop app via Tauri v2 (Rust). The Tauri layer only adds an *optional* native
integration (direct read/write of slicer profile files on disk); the web app is fully
functional without it.

## Commands

```bash
npm run dev                  # vite dev server, http://localhost:5173
npm run build                # tsc --noEmit, then vite build to dist/
npm test                     # vitest run (all suites)
npm run test:watch           # vitest watch mode
npx vitest run tests/formulas.test.ts   # single suite
npx vitest run -t "test name pattern"   # single test by name

npm run generate:printers    # regenerate src/data/printers.json from the xlsx workbook
npm run validate:printers    # CI check: fails if printers.json is stale vs. the workbook

npm run tauri dev            # desktop app in dev mode (needs Rust toolchain)
npm run tauri build          # native desktop build
```

Rust side (`src-tauri/`): tests are inline `#[cfg(test)]` modules in `backup.rs`,
`install.rs`, `discovery.rs` — run with `cargo test` from `src-tauri/`.

There is no lint script; TypeScript's `strict` mode (via `tsc --noEmit` in `npm run build`)
is the only enforced static check.

## Architecture

### Web app (`src/`) — always available, no native dependency

- `types.ts` — all domain types (printer profiles, calibration ids, etc.)
- `app.ts` / `main.ts` — shell, hash-based router (`#/wizard/:id/:step` etc.), theme, leave-guard
- `data/` — mostly-static content treated as data, not code:
  - `calibrations.ts` — the 7 calibration test definitions
  - `slicers.ts` — **version-aware** per-slicer instructions (Orca 2.4.x, Bambu 1.7+), each
    entry carries a `verifiedOn` date. Updating for a new slicer release means editing one
    data entry here, not code.
  - `materials.ts` — material presets (suggestions only, always editable)
  - `printerDatabase.ts` / `printers.json` — see "Printer database" below
  - `glossary.ts`, `models.ts` (external 3D model manifest)
- `logic/` — pure calculation/validation, decoupled from UI:
  - `formulas.ts` — the formula engine; every calculation returns
    inputs/formula/result/warnings (no black-box numbers anywhere in the UI)
  - `ranges.ts` — suggested test ranges derived from material + printer + extruder
  - `validation.ts`, `confidence.ts`, `recommendations.ts`
- `storage/` — IndexedDB wrapper + repository (`db.ts`), settings/drafts (`store.ts`)
- `export/backup.ts` — JSON export/import with schema versioning and migration
- `ui/` — one module per view/screen (dashboard, printers, project, wizard, forms, report,
  card, settings…), built on a tiny custom `h()`/DOM helper (`ui/dom.ts`) — no framework

**Adding a calibration test** = new entry in `data/calibrations.ts` + a form controller in
`ui/testForms.ts` + slicer steps in `data/slicers.ts`. No page redesign needed.

### Slicer integration (`src/slicerIntegration/`) — desktop-only, degrades to no-op on web

This subsystem lets the desktop app read/write Orca-family slicer profile files directly on
disk (scan installed slicers, install a generated preset, back up/restore user presets before
overwriting them). It is optional and gated behind `isDesktop()`.

- `bridge.ts` is **the single boundary** between web-safe code and native Tauri commands —
  everything else in this directory must go through it, so the browser/PWA build degrades
  cleanly to export-only behavior. It calls through `window.__TAURI__` (no Tauri npm
  dependency in the frontend bundle). Command names/payload shapes here must mirror
  `src-tauri/src/slicer_integration/` (serde snake_case on the Rust side, camelCase args here).
- `registry.ts` / `orcaFamily.ts` / `adapters/` — per-slicer adapters (Orca, Bambu, Elegoo
  Slicer, FlashPrint/FlashStudio, Snapmaker Orca) describing profile locations and formats
  for the Orca-derived slicer family.
- `scanner.ts`, `generator.ts`, `diff.ts`, `validation.ts`, `installer.ts`,
  `recommendations.ts`, `diagnostics.ts`, `errors.ts` — scan existing profiles, generate a
  new preset from wizard results, diff/validate before writing, and produce the install plan.
- `libraryBackup.ts`, `featureFlags.ts` — experimental features are gated by flags persisted
  under a separate localStorage key (`perfectfit.experimentalFeatures`) so they never touch
  `AppSettings` or its backups.
- Fixtures for this subsystem's tests live in `tests/slicerIntegration/fixtures/` — real
  sampled profile JSON/info files from each supported slicer, used to test adapters/scanner/
  generator against actual on-disk formats rather than synthetic data.

### Desktop shell (`src-tauri/`)

Rust/Tauri v2 backend. `src/slicer_integration/` (`discovery`, `filesystem`, `processes`,
`backup`, `install`, `security`) implements the native commands that `bridge.ts` calls —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Django1982/Filament_Calibration_Wizard](https://github.com/Django1982/Filament_Calibration_Wizard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
