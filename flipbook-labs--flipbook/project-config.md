---
trigger: always_on
description: Flipbook is a Roblox Studio plugin for browsing and testing UI stories. The dependency chain is:
---

# Flipbook - Agent Instructions

## Repo Overview

Flipbook is a Roblox Studio plugin for browsing and testing UI stories. The dependency chain is:

```
Flipbook  ←  Storyteller  ←  ModuleLoader
```

All three repos (`flipbook`, `storyteller`, `module-loader`) use the same toolchain and conventions, and are expected to be checked out as siblings on disk (e.g. `~/git/flipbook`, `~/git/storyteller`, `~/git/module-loader`).

---

## Tech Stack

| Tool                     | Role                                                                           |
| ------------------------ | ------------------------------------------------------------------------------ |
| **Lute**                 | Task runner for all scripts (`lute run <script>`)                              |
| **Rokit**                | Toolchain version manager (`rokit install` pins tools from `rokit.toml`)       |
| **Wally**                | Roblox package manager (Roblox runtime deps)                                   |
| **Loom**                 | Luau package manager for tooling/scripts (installs to `LuauPackages/`)         |
| **Rojo**                 | Syncs Luau source trees to Roblox place format; used for sourcemaps and builds |
| **Darklua**              | Transforms Luau-style `require` paths to Roblox `require()` calls during build |
| **Rocale**               | Roblox Open Cloud CLI; uploads and runs test places in the cloud               |
| **Jest** (jsdotlua)      | Unit test framework; tests run inside a Roblox place via Rocale                |
| **Selene**               | Luau linter                                                                    |
| **StyLua**               | Luau formatter                                                                 |
| **React** (17, jsdotlua) | UI framework used throughout Flipbook and Storyteller                          |
| **Charm**                | Reactive signals library; used for stores in both Flipbook and Storyteller     |
| **ModuleLoader**         | Bypasses Roblox's require cache; core to how Flipbook reloads stories          |
| **Storyteller**          | Story discovery, loading, and rendering; wraps ModuleLoader                    |

---

## Repository Layout

### Flipbook

```
flipbook/
├── src/                    # Thin plugin entry point (Studio bootstrap only)
├── workspace/              # Real application code, organized as workspace members:
│   ├── flipbook-core/      # Main library — React app, story browser, telemetry, settings
│   ├── flipbook-next/      # Experimental next-gen package
│   ├── test-runner/        # Runs Jest.runCLI against flipbook-core tests
│   ├── example/            # Dogfood stories/components
│   ├── code-samples/       # Sample stories for Roact, Fusion, React+Storyteller
│   └── template/           # Scaffold for new workspace members
├── build/                  # Build output (gitignored):
│   ├── dev/roblox/         # Dev plugin build
│   ├── prod/roblox/        # Production build
│   └── flipbook-core-rotriever/  # Rotriever bundle for Studio-internal Flipbook
├── .lute/                  # Lute task scripts
├── Packages/               # Wally installs (gitignored)
├── LuauPackages/           # Loom/Lute tooling packages (moved from Packages/ on install)
├── RobloxPackages/         # roblox-packages CLI installs (Foundation, Promise, Dash, etc.)
├── project.luau            # Shared path constants used by all Lute scripts
├── wally.toml              # Roblox runtime dependencies
├── loom.config.luau        # Loom manifest (Lute, flipbook-batteries, dotenv)
└── .env / .env.template    # Environment variables (copy template to .env)
```

Most application code lives under `workspace/flipbook-core/src/`, not `src/`. The root `src/init.server.luau` is only a thin bootstrap that calls `FlipbookCore.createFlipbookPlugin(...)`.

### Storyteller / ModuleLoader

Both follow a simpler layout:

```
<repo>/
├── src/                    # Authoring source
├── dist/                   # Build output (gitignored) — what Wally ships
├── .lute/                  # Lute task scripts
├── Packages/               # Wally installs
├── LuauPackages/           # Loom tooling packages
├── project.luau            # Shared path constants
├── wally.toml              # Package metadata + dependencies
└── loom.config.luau        # Loom manifest
```

Build output (`dist/`) is a Darklua-processed mirror of `src/` with Luau-style requires converted to Roblox `require()` calls. **Wally publishes from `dist/`**, not `src/`.

---

## Scripts (`lute run`)

Lute is the task runner for all three repos — the Luau equivalent of `npm run`. **Before reaching for any external tool or shell command, check whether a `lute run` script already covers it.** All scripts live in `.lute/<name>.luau` and are invoked as `lute run <name>`.

### Common Commands

```bash
# Flipbook — build dev plugin to Studio plugins folder
lute run build plugin --channel dev

# Flipbook — full rebuild (use after dependency changes)
lute run build plugin --channel dev --clean

# Flipbook — watch mode (incremental on workspace member changes)
lute run build plugin --channel dev --watch

# Flipbook — build FlipbookCore as a rotriever bundle for local integration flows

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flipbook-labs/flipbook](https://github.com/flipbook-labs/flipbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
