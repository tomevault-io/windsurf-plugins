---
trigger: always_on
description: Operating instructions for coding agents working in this repository.
---

# AGENTS.md

Operating instructions for coding agents working in this repository.

## Project identity

- Project: `desktop-nano-cortex` — "Desktop Nano Cortex"
- Purpose: unofficial desktop companion for the Neural DSP Nano Cortex. Not affiliated with or endorsed by Neural DSP.
- Stack: Tauri 2 desktop app — Rust backend (`backend/`, the `src-tauri` equivalent) + React 19 / TypeScript / Vite 7 / Tailwind 4 frontend (`frontend/`)
- Package manager: npm. This is a **two-package repo, not a workspace**: root `package.json` orchestrates (and owns the DX tooling); `frontend/package.json` owns the app deps. Each has its own lockfile.
- Rust: 1.77.0 (pinned in `backend/rust-toolchain.toml`). Node: 20+ (`.nvmrc`, `engines`).

## Product truth (read this first)

This project's value is honesty about what the device actually supports:

- **Documented MIDI is authoritative.** Outgoing Program Change / Control Change and observed incoming USB MIDI are the trusted surface (zone `100`).
- **BLE state is provisional.** BLE private-protocol decoding (zone `110`) is reverse-engineered and experimental. It must always be labelled as such in UI, docs, and release notes.
- **Forbidden unless hardware-verified and spec-linked:** claims of a full editor, preset sync, parameter sync, or authoritative BLE state.
- Allowed language: "Documented MIDI", "Observed incoming MIDI", "Experimental BLE diagnostics", "Provisional BLE decode", "Hardware verification required".

Full guard: `docs/specs/900-project-governance/spec.md` (Truthfulness Guard appendix).

## Commands

Run from the repository root.

```bash
npm install                    # root tooling (husky, knip, prettier, commitlint, …)
npm ci --prefix frontend       # frontend deps
npm run dev                    # Vite dev server (browser, mocked backend)
npm run dev:tauri              # full desktop app (Rust backend + webview)
npm run build:mac              # local dmg/app bundle (macOS host)
```

`just` mirrors the surface (`just setup`, `just check`, `just fix`, `just verify`, `just verify-full`, …).

## Verification

Two-tier surface; the pre-push hook runs the fast tier.

```text
npm run check        Static checks, no mutation:
                     format:check · lint (ESLint + clippy -D warnings + traceability gate)
                     · typecheck · knip · lint:md
npm run verify       check + unit tests (Vitest + cargo test). Fast gate — use after every change.
npm run verify:full  verify tier + coverage + e2e + frontend build + dependency/license
                     checks + version drift. Use before merging. What CI (manual dispatch) runs.
```

The loop: `npm run verify` → if it fails on mechanical issues → `npm run fix` (prettier + rustfmt →
markdownlint --fix → eslint --fix) → `npm run verify` again. `fix` does NOT resolve type errors,
knip findings, traceability gaps, or commitlint failures — those need real changes.

**Hardware rule:** changes to MIDI/BLE behavior additionally require the manual smoke runbook
(`docs/runbooks/hardware-smoke.md`) against a real Nano Cortex — CI has no hardware.

**Never report a task complete without running `npm run verify` and reading the output.**

## Commit log conventions

Enforced by husky + commitlint (see `commitlint.config.cjs`):

- Header: `type(scope): imperative summary` — scope is **required**, from:
  `overview, midi-usb, ble, ipc, platform, frontend, dx, ci, release, governance, deps, docs, repo, scripts, spec, security`. Max 100 chars.
- Committing on `main` is allowed (solo-dev policy). Feature branches (`feat/…`, `fix/…`, `chore/…`) are encouraged; non-conforming branch names get a warning, not a block.
- Non-trivial commit bodies use the AFX shape (template in `.gitmessage`):

```text
Why:
- What problem this solves.

Changed:
- What changed, grouped by surface.

Spec:
- docs/specs/XXX-name/spec.md [FR-X]

Verification:
- npm run verify
```

Use `docs(spec)` for spec/design/tasks-only changes, `feat`/`fix` only for behavior changes,
`refactor` only when behavior is intentionally unchanged.

## Layout rules

```txt
backend/     Rust crate: MIDI (midir), BLE (btleplug, behind the default `ble` feature),
             IPC commands, tray/shortcuts/store. Owns tauri.conf.json, capabilities/.
frontend/    React app: features/midi components + hooks, shared/ipc wrappers, e2e/ Playwright.
docs/specs/  AFX zones (spec.md / design.md / tasks.md per zone) — the source of truth.
docs/runbooks/  Manual hardware smoke, release checklist, Windows build.
scripts/     Root utility scripts (traceability gate, version sync, license check).
```

- Device I/O lives in Rust only. The webview sends documented commands and renders observed/provisional state.
- Frontend code must not import `@tauri-apps/api` directly in unit-testable modules — go through the wrappers in `frontend/src/shared/ipc/` so tests can mock without patching globals.
- Vitest must run without MIDI hardware or a Tauri runtime; Playwright drives Chromium against the Vite dev server with Tauri IPC mocked (`frontend/e2e/fixtures/tauri-mock.ts`).

## Generated files and artifacts

Do not edit manually: `node_modules/`, `backend/target/`, `backend/gen/`, `frontend/dist/`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rixrix/deskop-nano-cortex](https://github.com/rixrix/deskop-nano-cortex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
