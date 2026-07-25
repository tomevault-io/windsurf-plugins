---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

Babylon.js Editor 5 — a desktop (Electron) application for creating and editing 3D scenes with the Babylon.js engine. This is a **yarn classic workspaces monorepo**. The workspaces are:

- `editor/` — `babylonjs-editor`: the Electron app (main process + React renderer). Also publishes a public API (`editor/src/export.ts`) consumed by plugins/scripts.
- `tools/` — `babylonjs-editor-tools`: a runtime library bundled into exported/packaged projects (scene loading, decorators, rendering pipelines, cinematic system). This is what user game code (`src/scripts.ts` etc.) imports.
- `cli/` — `babylonjs-editor-cli`: command-line tool used to pack/export a project's assets, geometries, scenes and scripts, plus S3 upload helpers.
- `mcp/` — `babylonjs-editor-mcp-server`: MCP server exposing editor scene-composition capabilities to AI agents (see `mcp/specifications.md` for the product/feature spec).
- `plugins/` — editor marketplace plugins (`quixel`, `fab`).
- `templates/` — project templates scaffolded for new user projects: `nextjs`, `nuxtjs`, `solidjs`, `vanillajs`, `electron`.
- `website/` — Next.js marketing + documentation site (editor.babylonjs.com).

The `@babylonjs/*` / `babylonjs-*` engine packages are pinned via the root `resolutions` field (currently `9.9.1`) — keep these in sync if bumping the engine version.

## Common commands

Install once at the root (yarn classic):
```bash
yarn install
```

### Build
```bash
yarn build              # tools -> cli -> mcp-server -> editor -> plugins (sequential, dependency order)
yarn build-all          # build + templates + website
yarn build-all-concurrently
yarn build-editor / build-tools / build-cli / build-mcp-server / build-plugins / build-templates / build-website
```

### Run / develop
```bash
yarn start                 # launches the Electron editor (babylonjs-editor)
yarn watch-editor-all       # tsc + esbuild + tailwind watch for the editor
yarn watch-tools / watch-cli / watch-mcp-server / watch-plugins
```
In VS Code, `Cmd/Ctrl+Shift+B` → `watch-all-editor` runs all watchers concurrently (editor, tools, plugins, cli, mcp-server). When changing `tools/`, `cli/`, or `mcp/`, run their watcher alongside the editor's so the editor picks up the rebuilt dependency.

### Lint & format
```bash
yarn format / yarn format-check     # prettier (tabs, double quotes, printWidth 180, trailing comma "es5")
yarn lint                           # format-check + eslint across editor, tools, cli, mcp-server, plugins, templates, website
yarn lint-fix                       # format + eslint --fix across all of the above
yarn lint-editor / lint-tools / lint-cli / lint-mcp-server / lint-plugins / lint-templates / lint-website
```
ESLint config is the flat config at `eslint.config.mjs` plus the custom rule `.eslint/eslint-rules/require-return-type-on-class-methods.mjs` (every class method needs an explicit return type). `editor/src/ui/shadcn/**` and template `scripts.ts` files are excluded from linting.

### Tests
Tests use vitest, per-workspace:
```bash
yarn test                                                  # tools tests then editor tests
yarn coverage                                              # same, with coverage
yarn workspace babylonjs-editor-tools test                 # tools/test/**/*.test.ts
yarn workspace babylonjs-editor test                       # editor/test/**/*.test.mts
```
To run a single test file, `cd` into the workspace and call vitest directly, e.g.:
```bash
cd tools && yarn vitest run test/tools/scene.test.ts
cd editor && yarn vitest run test/tools/node/clone.test.mts
```
`cli/` and `mcp/` currently have no test scripts.

### Packaging
```bash
yarn package --noSign [--x64] [--arm64]   # full pipeline: clean, install, lint, build-all-concurrently, test, then build.mjs (electron-builder)
```
Packaging is platform-bound: macOS builds must run on macOS, Windows builds on Windows (native deps, code signing). macOS signing/notarization needs `APPLE_ID`, `APPLE_APP_SPECIFIC_PASSWORD`, `APPLE_TEAM_ID` in a root `.env`.

## Architecture

### Electron app (`editor/`)
- `editor/src/index.ts` — Electron **main process** entry point. Manages the dashboard window vs. per-project editor windows (`editorWindows`), app menu setup, IPC routing (`ipcMain`), auto-update, and registers the `babylonjs-editor://` protocol. Side-effect imports wire up IPC handlers: `electron/node-pty`, `electron/events/{shell,dialog,editor,window,export}`, `electron/assimp/assimpjs`, `electron/protocol`, `electron/oauth`.
- `editor/src/dashboard/` — the project-picker/dashboard window (renderer + its own preload/menu).
- `editor/src/editor/main.tsx` — the `Editor` React component, the **renderer** root for a project window. Holds top-level state (compressed-texture settings, opened tabs, plugin list) and reacts to IPC events (`save`, `editor:open`, `editor:run-project`, undo/redo, etc.).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BabylonJS/Editor](https://github.com/BabylonJS/Editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
