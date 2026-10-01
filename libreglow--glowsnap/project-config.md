---
trigger: always_on
description: Guidance for AI coding agents working in this repository. Before changing
---

# AGENTS.md — GlowSnap

Guidance for AI coding agents working in this repository. Before changing
anything, inspect the relevant code and configs. Make the smallest focused
change that preserves the existing architecture.

## Project Overview

GlowSnap is an open-source, **Linux-only** desktop app for screen capture,
screen recording, and visual editing. It is a Wails v2 application: a Go
backend handles native Linux integration (Screenshot/ScreenRecorder D-Bus
portals, system tools), and an embedded React + TypeScript UI renders inside a
WebKitGTK window. Screenshot editing happens on a Konva canvas.

## Tech Stack

- **Backend:** Go 1.25, Wails v2 (pinned `v2.13.0`), `godbus/dbus/v5`,
  `golang.org/x/sys`.
- **Frontend:** React 19, TypeScript 5.9 (strict), Vite 8, Tailwind CSS v4,
  shadcn/ui (`base-nova`, backed by `@base-ui/react`), Konva + react-konva,
  framer-motion, lucide-react.
- **Package manager:** npm is canonical. `frontend/package-lock.json` is the
  authoritative lockfile; CI and the Dockerfile use `npm ci`. A stale `bun.lock`
  exists but is not the source of truth.
- **Formatting:** gofmt for Go (enforced in CI via `gofmt -l .`). No frontend
  formatter or linter is installed.

## Repository Structure

- `main.go` — entry point; embeds `frontend/dist` and `build/appicon.png` via
  `//go:embed`; configures `wails.Run`.
- `app.go` — the Wails-bound `App` type: the **bridge/orchestration layer**
  between the frontend and the backend services.
- `services/` — plain Go packages (`screenshot`, `screencast`, `settings`);
  `ocr/` is empty. No Wails dependency.
- `frontend/` — React/TS app: `src/` (components, hooks, lib, types),
  `index.html`, `tsconfig.json`, `vite.config.ts`, `vitest.config.ts`,
  `components.json` (shadcn config), and `wailsjs/` (generated bindings).
- `scripts/` — `dev.sh`, `build.sh`, `build-appimage.sh`, `release.sh`,
  `install.sh`, `clean.sh`, `install-deps.sh` (docs in `scripts/README.md`).
- `build/` — app icon, `glowsnap.desktop`, `icons/hicolor` theme; `bin/` and
  `AppImage/` hold build output.
- `packaging/` — Flatpak manifest, desktop file, icon, metainfo.
- `Dockerfile` — reproducible multi-stage Linux build + test environment.
- `.github/workflows/` — `ci.yml` (backend + frontend checks) and `release.yml`.
- `third_party/wails` — vendored/local fork of the Wails module, wired via
  `replace github.com/wailsapp/wails/v2 => ./third_party/wails`.
- `docs/`, `README.md`, `CONTRIBUTING.md`.

## Architecture

The intended data flow is:

```text
React frontend
      ↓
Generated Wails bindings (frontend/wailsjs)
      ↓
App methods in app.go
      ↓
Services / native Linux integrations (services/)
```

The frontend calls backend methods through the generated bindings in
`frontend/wailsjs/go/main/App`; backend → frontend events flow through
`runtime.EventsEmit` / `EventsOn` (e.g. `recording-started`,
`recording-ended`, `toggle-palette`). Do not bypass this architecture without a
clear reason.

- **Window modes** are a union type (`palette | studio | closed | recording |
  settings | preferences | overlay`) managed in `App.tsx` through local state
  and passed to components as props. Corresponding `ResizeTo*` methods on `App`
  resize the native window.
- **Capture** uses the org.freedesktop.portal Screenshot API over D-Bus in
  `services/screenshot`; **recording** uses the ScreenRecorder portal plus a
  GStreamer pipeline in `services/screencast`.
- **Settings** live in `services/settings`: JSON at
  `~/.config/glowsnap/settings.json`, loaded with defaults-merge +
  normalization, saved atomically via a `.tmp` file + rename.
- **Screenshots** saved to the settings save-directory are served to the UI by
  a loopback `127.0.0.1` HTTP server started in `App.startup`; the frontend
  loads image URLs from `GetScreenshotsBaseURL()`.
- **Editor** renders a Konva stage driven by the `ShapeConfig` model
  (`src/types/types.ts`), with history via `useHistory`.

### app.go vs services/

- `app.go` should remain primarily the Wails bridge/orchestration layer. Avoid
  dumping unrelated business logic into it.
- Put reusable functionality in the appropriate `services/` package.
- `services/` packages must stay plain Go and must **not** depend on Wails
  runtime APIs. Wails-specific concerns belong in `app.go` / `main.go`.
- Do not duplicate service functionality inside `app.go`.
- Add an exported method to `App` only when functionality genuinely needs to be
  exposed to the frontend; let Wails regenerate the bindings rather than
  editing `frontend/wailsjs` by hand.
- Keep services independently testable where practical, and do not introduce
  abstractions the current project does not need.

## Change Scope

- Inspect the relevant implementation before changing it.
- Identify the files directly related to the task and touch only those.
- Make the smallest reasonable change.
- Avoid unrelated cleanup and unrelated refactoring.
- Do not rename or reorganize files unless required.
- Do not replace working libraries or architectural patterns without a clear
  reason.
- Do not modify unrelated configuration.
- Do not remove existing functionality unless explicitly requested.
- Preserve existing behavior outside the requested change.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [libreglow/glowsnap](https://github.com/libreglow/glowsnap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
