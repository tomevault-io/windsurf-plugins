---
trigger: always_on
description: These notes are curated for downstream agents. Treat them as binding guidance: each section funnels a different expectation for builds, style, testing and compliance with upstream tooling. (`CLAUDE.md` is a symlink to this file.)
---

# AGENT OPERATING NOTES

These notes are curated for downstream agents. Treat them as binding guidance: each section funnels a different expectation for builds, style, testing and compliance with upstream tooling. (`CLAUDE.md` is a symlink to this file.)

## 1. Project layout recap

A Wails app: Go backend + Svelte (SvelteKit) frontend, packaged as a single native binary.

- `frontend/` — Bun + SvelteKit UI written in **Svelte 5 (runes)**. Built by Vite into `frontend/build` and embedded into Go via `//go:embed all:frontend/build` in `main.go`. Wails TS bindings are generated into `frontend/src/lib/wailsjs`.
- `api/` — the Wails-bound `Controller`: install/repair/uninstall actions, native dialogs, and the GUI logging bridge.
- `betterdiscord/` — download plus install/repair/uninstall of the BetterDiscord asar.
- `discord/` — Discord install discovery and injection; per-OS path logic lives in `paths_*.go`.
- `types/`, `utils/`, `wsl/` — shared Go types, backend helpers, and WSL detection/path mapping.
- `main.go` / `app.go` — Wails entry point, window options, and the startup update check.
- `build/` — product icons plus plist/manifest/packaging templates (scaffolded/generated; not usually hand-edited).
- `scripts/` — release helpers: `build-frontend.sh`, `build-appimage.sh`, `render-plist.sh`, and the `winres` Go generator.
- `wails.json` — Wails config + frontend hooks (`frontend:install` = `bun install`, `frontend:build` = `bun run build`). Keep it synchronized with the commands you run locally.
- `Taskfile.yml` — dev/build/check/release shortcuts. `.goreleaser.yaml` — release build + publishing config.

## 2. Build, dev & release commands

Prereqs: Go (matches `go.mod`), Bun, the Wails CLI, and optionally Task + GoReleaser. On Linux, `wails dev`/`wails build` require `-tags webkit2_41` (the Task shortcuts add this automatically).

Follow this checklist before you claim a build is validated.

1. `cd frontend && bun install`
   * Bootstraps the renderer toolchain. Required before `wails dev/build` and before running any Bun scripts. The `frontend:install` hook in `wails.json` runs this during `wails build`.
2. `task dev` (or `wails dev`)
   * Launches the Go server, compiles the renderer on demand, and proxies events between them. Closest analog to the installed app; use it for interactive testing (dialogs, runtime events, system prompts).
3. `task build` (or `wails build`)
   * Produces a local native binary and triggers the `frontend:build` hook (`bun run build`) so the embed picks up the latest UI. Note: official release artifacts come from GoReleaser (see §6), not `wails build`.

### Frontend-only helpers (run from `frontend/`)

- `bun run dev` – Vite dev server only (renderer, no Go bindings).
- `bun run build` – rebuild the renderer bundle into `frontend/build` when changing UI/TS/CSS without touching Go.
- `bun run check` – `svelte-kit sync` + `svelte-check` (Svelte/TypeScript static checks).
- `bun run lint` – ESLint using the shared config in `frontend/eslint.config.js`.
- `bun run test` – Vitest unit tests.

### Go-specific commands

- `go test ./...` (or `task test`) – backend tests.
- `go vet ./...` (or `task vet`) – static analysis.
- `gofmt -w .` – required formatting for the `.go` files you touch.
- `task check` – runs frontend check + lint, `go vet`, and `go test` in one shot. It does **not** run the frontend Vitest suite; run `bun run test` for that.

## 3. Running isolated tests

- Backend: narrow with the `-run` regex, e.g. `go test ./types -run TestDiscordChannel_Name` (swap the suite name as needed).
- Many backend tests gate on `runtime.GOOS` with `t.Skipf` when the OS doesn't match — run the subset aligned with your OS; these are guards, not cross-platform stubs.
- The frontend has **real Vitest unit tests** (co-located `*.test.ts`, e.g. `Checkbox.test.ts`, `handlers.test.ts`). Run all with `bun run test`, or a single file with `bunx vitest run src/lib/utils/handlers.test.ts`. `bun run check` covers types separately.

## 4. Style rules – Go / backend

1. **Imports**
   * Standard library imports first (e.g., `context`, `embed`, `log`).
   * Blank line, then external dependencies (Wails, pkg/browser), then another blank line.
   * Local packages (`installer/api`, `installer/types`, `installer/utils`, …) go last.
2. **Formatting**
   * Always run `gofmt`. Tabs are canonical Go indentation.
   * Tables (like the ones in `channel_test.go`) are aligned for readability.
3. **Types & naming**
   * Exported structs and interfaces follow PascalCase (e.g., `App`, `Controller`).
   * Private helpers stay lowercase.
   * Channel constants are grouped and defined with `iota` (`Stable`, `Canary`, `PTB`).
4. **Error handling**
   * Prefer early returns instead of nested conditionals.
   * When something fails (e.g., `install.InstallBD()`), emit the failure event (`runtime.EventsEmit(action.ctx, "failure")`) and `return` immediately.
   * Swallow errors only when a retry loop is not viable; always log or emit events so the renderer can surface the failure.
5. **Context & runtime**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BetterDiscord/Installer](https://github.com/BetterDiscord/Installer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
