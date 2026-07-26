---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Quest Viva is an open-source system for creating and playing text adventure games. It's a .NET 10.0 C# application, the modern successor to Quest 5. There are two player implementations: **WebPlayer** (ASP.NET Core + Blazor Server, Docker-deployed) and **WasmPlayer** (pure browser-WASM, AOT-compiled, no server required — the long-term direction).

## Build & Test Commands

```bash
# Build (full solution)
dotnet build --configuration Release

# Run all tests
dotnet test --configuration Release

# Run a single test project
dotnet test tests/EngineTests

# Run a specific test
dotnet test tests/EngineTests --filter "FullyQualifiedName~TestMethodName"

# Run WebPlayer with Docker
docker compose up --build    # WebPlayer on http://localhost:8080

# Build WasmPlayer (Debug — fast interpreter mode)
dotnet build src/WasmPlayer/WasmPlayer.csproj

# Build WasmPlayer (Release — AOT compiled, ~15s)
dotnet build --configuration Release src/WasmPlayer/WasmPlayer.csproj

# Run WasmPlayer dev server
node src/WasmPlayer/dev-server.mjs              # Debug build
node src/WasmPlayer/dev-server.mjs --release    # Release/AOT build
# Open: http://localhost:5175/?url=/examples/simple.aslx
```

Tests use MSTest with Moq (mocking) and Shouldly (assertions).

## Documentation Site (site/)

Separate Astro/Starlight project for the documentation website:

```bash
cd site
npm install
npm run dev       # Dev server
npm run build     # Production build
npm run lint      # ESLint
```

## Architecture

The solution (`QuestViva.sln`) has a layered architecture:

```
WebPlayer (ASP.NET Core + Blazor Server)  ─┐
WasmPlayer (browser-wasm, AOT)             ─┤
                                            ├─► PlayerCore ─► Engine ─► Common
EditorCore ─────────────────────────────────┘        │
                                                     └─► Legacy
```

**Key projects in `src/`:**

- **Common** — Shared types and interfaces used across all projects
- **Engine** — Core game interpreter: script execution, expression evaluation, game loading, built-in functions. Contains embedded `.aslx` files (game templates, language definitions) in `Core/`
- **PlayerCore** — Game player runtime that wraps Engine. Contains embedded UI resources (HTML, CSS, JS including jQuery UI, jPlayer)
- **EditorCore** — Game editor logic (non-UI)
- **Legacy** — Quest 4 (and earlier) backward-compatibility layer with embedded `.lib`/`.dat` files
- **WebPlayer** — ASP.NET Core web app with Blazor Razor components (`Game.razor`, `Slots.razor`, debugger)
- **WasmPlayer** — Pure browser-WASM player (`browser-wasm` target, AOT-compiled). Uses `JSImport`/`JSExport` for JS interop. Serves as a static site with no server-side .NET required. IL trimming is enabled; `WasmPlayer.linker.xml` preserves the Engine assembly (which uses reflection-based type discovery).
- **WasmEditor** — Browser-WASM bridge (`browser-wasm` target) exposing `EditorCore` to the AppShell SvelteKit frontend via `[JSExport]` (see `WasmEditorBridge.cs`)
- **AppShell** (`src/AppShell/`) — SvelteKit SPA (adapter-static) frontend for the game editor; talks to WasmEditor over the JS/WASM boundary and to `FileAdapter` implementations (`src/lib/filesystem/`) for storage (FSA, OPFS local drafts, server, Electron). Also serves the Play/Create Home landing page at root when `PUBLIC_SHOW_HOME=true` (play.questviva.com, Electron) — root shows a game catalog (Play tab, fetched from textadventures.co.uk's `api/Catalog`) or the editor canvas once a game is loaded; `/open` (Create tab) is unchanged; `/play/[id]` is a new game-detail route. Unset (textadventures.co.uk) keeps the previous editor-only root behavior. See `docs/appshell-wasm-svelte.md` and `docs/deployment-domains.md`
- **ElectronApp** (`src/ElectronApp/`) — Electron main-process shell (desktop app) wrapping the AppShell SPA over a local loopback HTTP server; no Svelte/UI code of its own. See `docs/electron-desktop-app.md`

**Test projects in `tests/`:** EngineTests, PlayerCoreTests, EditorCoreTests, UtilityTests, LegacyTests

## Git Workflow

`main` is a protected branch (required status check `build_and_test`, required PR review, `enforce_admins` on) — direct pushes are rejected outright, including from repo admins. All changes, however small, go through a feature branch + PR.

## Releasing

Releases are managed by [release-please](https://github.com/googleapis/release-please) (`.github/workflows/release-please.yml`, config in `release-please-config.json` / `.release-please-manifest.json`). There's no manual `VERSION`-bump PR:

1. PRs must have a [Conventional Commits](https://www.conventionalcommits.org/)-prefixed title (`fix:`, `feat:`, `chore:`, etc.) — enforced by `pr-title-lint.yml`, which also restricts the optional scope (e.g. `feat(AppShell): ...`) to an exact-case allowlist of project names, so scoped changelog entries cluster and capitalize consistently. Since PRs are squash-merged, the PR title becomes the commit message on `main` that release-please parses.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [textadventures/quest](https://github.com/textadventures/quest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
