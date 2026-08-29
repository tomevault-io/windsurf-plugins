---
trigger: always_on
description: > **Boundary:** standalone desktop app; no shared DB or API with the SpaceVibe web repos.
---

# AGENTS.md — SpaceVibe Deck

> **Boundary:** standalone desktop app; no shared DB or API with the SpaceVibe web repos.
> Do not edit sibling repos from this session. Workspace map:
> [`../AGENTS.md`](../AGENTS.md) `current`.

Deck is a terminal for running many agent CLIs side by side. `main` carries **two hosts**: the
Tauri 2 + Rust host that every release still builds, and the Electron host in `electron/` that
is meant to replace it. The renderer is Preact + xterm.js and reaches whichever host it runs
under through the facades in `src/host/`. Everything in this repo — UI strings, comments, docs,
and commits — is **English only**.

Project state: [docs/CONTEXT.md](docs/CONTEXT.md) `current`; architecture:
[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) `current`; visual rules:
[docs/DESIGN-LANGUAGE.md](docs/DESIGN-LANGUAGE.md) `current`.

## Current direction

- **Auto-update is a core requirement.** A release is not complete if distribution falls
  back to manual-download-only. Release claims require platform-specific runtime evidence.
- **Tauri is feature-frozen** except hotfixes and release support. New product features land
  on Electron so they are not implemented twice.
- **Tags ship Electron on both platforms now; Tauri is retired from tag triggers
  (2026-08-20).** [`electron-release.yml`](.github/workflows/electron-release.yml) `current`
  is four jobs (prepare → mac + windows → promote): one `build/vX.Y.Z` (stable) or
  `build/vX.Y.Z-electron.N` (prerelease) tag — its commit must sit on `main` — produces a
  draft that goes public only with all six updater assets present, and a stable release
  publishes the tagged commit's `CHANGELOG.md` section as its notes (missing section = the
  release stays a draft); `release.yml` is `workflow_dispatch`-only for Tauri hotfixes. The
  stable is **`1.0.0`** — the owner named it a V1 (2026-08-20).
  **Gate A is CLOSED for macOS** — owner-verified auto-update against `v0.12.5-electron.2`,
  2026-08-19. Windows ships **unsigned and runtime-unverified by owner decision** (Gate C
  stays open), and an updating Windows `Deck Electron` preview.2 becomes a side-by-side
  `SpaceVibe Deck` install with fresh userData (fork F1's identity). No macOS preview ever
  shipped publicly, so the stable is the first public macOS release. **`SpaceVibe Deck 1.0.0`
  is PUBLIC and is `releases/latest` since 2026-08-20** — run 32383647050, all four jobs
  green, eight assets served; the maiden run before it died in `promote` on a
  transitive-needs empty output and published NOTHING, which is the fail-closed design
  working. See [spec](docs/specs/2026-08-20-electron-stable-release-design.md) `decided` and
  [plan](docs/plans/2026-08-20-electron-stable-release.md) `building`.
- The Electron cutover is a **clean install** with no settings/workspace migration. The final
  Tauri release must explain the manual transition and old data location. “No Electron” must
  stop being a proof point at cutover; “no accounts” remains valid. **“No telemetry” is
  retired, and analytics is ON by default (decided 2026-08-23, committed 2026-08-24 as
  `cdc07a0`):** the 2026-08-22 opt-in model was built, never released, and reversed by
  the owner the next day — no consent question is asked,
  `declined` (the Settings → Privacy switch) is the only state never inferred away, an
  unreadable state file still fails closed to off, and public copy says "on by default,
  no code, paths or prompts" and never "anonymous". `USAGE_CONSENT_ASKED` in
  [`usage-notice.ts`](src/telemetry/usage-notice.ts) `current` is the whole reversal
  switch; the consent modal stays in the tree behind it. Rollout consequence: every
  install of the next release POSTs, so the Worker and the privacy page are
  prerequisites, not follow-ups. See the
  [usage analytics spec](docs/specs/2026-08-22-anonymous-usage-telemetry-design.md)
  `decided` (amended 2026-08-24).
- Electron process classification must use the measured `ps` snapshot path, not
  `node-pty.process`; the latter returned version/executable strings instead of argv0.
- **Pane detach Phase A exists on Tauri**, including IPC contract tests; remaining native
  manual checks live in `docs/CONTEXT.md`. Phase B is Electron-only and still gated by a real
  Windows pointer-capture check.
- **The browser is a tab on the stage strip, not a docked column (2026-08-15).** One chip in
  the strip's second segment (globe + page title); its surface covers the stage like the
  document editor does (new DL-18.8), and the docked right column, its resize drag and the
  `browserWidth` setting are gone. [`composeSurfaceStrip`](src/ui/stage-surface-strip.ts)
  `current` folds it into TabManager's `SurfaceStrip` seam, so ⌘W, tab cycling and
  "last surface, not last tab" reach it without touching R4 seams. The `WebContentsView`
  itself, react-grab Inspect and `electron/browser/` are unchanged. Electron only; verified
  by suite/build only — no native `electron:dev` pass or owner eye review yet. No Tauri
  implementation exists; its behaviour under `npm run tauri dev` is unverified.
- **A grab stops at the clipboard and no longer reaches a pane (2026-08-16, temporary).**
  [`GRAB_PASTE_DISABLED`](src/browser/browser-store.ts) `current` short-circuits

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mxrsv/spacevibe-deck](https://github.com/mxrsv/spacevibe-deck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
