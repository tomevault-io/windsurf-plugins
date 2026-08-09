---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Overview

**brs-engine** is a BrightScript Simulation Engine: an interpreter for the BrightScript language that runs Roku apps (channels) in web browsers and Node.js. It simulates the BrightScript runtime, the Draw 2D API (`roScreen`, `roCompositor`, `roRegion`, …), the SceneGraph framework, the Roku file system, registry, remote control, and the Micro Debugger — targeting compatibility up to Roku OS 15. It is **not** a Roku OS or hardware emulator; it is a development/automation tool. The repo was originally forked from [rokucommunity/brs](https://github.com/rokucommunity/brs).

Node.js **v22 or newer** is required to build and to run the CLI.

## Monorepo layout

This is an npm **workspaces** monorepo (root package `brs-engine-workspace`). All TypeScript source lives in the top-level `src/` directory and is compiled into three published packages under `packages/`:

- **brs-engine** (`packages/browser`) — browser / Web Worker interpreter for web, PWA, and Electron apps. Build output: `packages/browser/lib/brs.api.js` + `brs.worker.js`, types in `packages/browser/types/`.
- **brs-node** (`packages/node`) — Node.js library plus the `brs-cli` command, ECP + SSDP servers. Build output: `packages/node/bin/{brs.cli.js, brs.ecp.js, brs.node.js}`.
- **brs-scenegraph** (`packages/scenegraph`) — SceneGraph runtime shipped as a standalone **extension** bundle (`brs-sg.js` / `brs-sg.node.js`) that auto-loads when an app contains `pkg:/components/` assets.

### Required deployment asset: `assets/common.zip`

`packages/browser/assets/common.zip` (and its SceneGraph-aware counterpart in `packages/scenegraph/assets/common.zip`) is the **`common:/` volume** — it contains the default fonts, system audio, CA certificates, and BrightScript library stubs (`LibCore`, `roku_ads`, `roku_analytics`, `roku_browser`) that all BrightScript apps expect to be present. **Any web app that embeds the engine must serve this file at `./assets/common.zip` relative to `brs.api.js`.** The API library fetches it automatically on startup via `fetch('./assets/common.zip')` — if the file is missing, fonts and system libraries will be unavailable and most apps will fail or look broken.

## Commands

Run from the repo root (scripts fan out to workspaces):

```bash
npm install              # install all workspace dependencies

npm run build            # dev build of all packages (--workspaces)
npm run build:api        # build only brs-engine (browser)
npm run build:node        # build only brs-node (CLI/Node library)
npm run build:sg         # build only brs-scenegraph
npm run build:web        # build engine + scenegraph, then open the example web app
npm run build:cli        # build Node + scenegraph
npm run release          # minified production build of all packages
npm run clean            # remove compiled lib/ bin/ types/ from all packages

npm start                # webpack-dev-server for the example web app (brs-engine)

npm run lint             # eslint over ./src
npm run prettier         # check formatting (4-space indent, printWidth 120)
npm run prettier:write   # auto-format

npm test                 # vitest (config in vitest.config.mts)
```

Tests live in `test/` (`brsTypes/`, `core/`, `interpreter/`, `lexer/`, `parser/`, `preprocessor/`, `stdlib/`, `extensions/`, `simulator/`, `cli/`). The e2e suite in `test/e2e/` is driven by `test/e2e/E2ETests.js`, comparing interpreter output against `.brs` fixtures in `test/e2e/resources/`. Test files are plain `.test.js`.

```bash
npx vitest run test/e2e/Functions.test.js   # run a single test file
npx vitest run -t "name of the test"        # run by test name
npx vitest run --update                     # refresh snapshots
```

After `npm run build:cli`, link the CLI for local use: `cd packages/node && npm link`, then `brs-cli`.

## Core architecture

### Two-thread split (browser model)

The browser build is two bundles that run on **separate threads** and communicate via `postMessage` + a shared `Int32Array` over `SharedArrayBuffer`:

- **API library** — entry `src/api/index.ts`, output `brs.api.js`. Runs on the **main thread**. Creates/manages the worker, renders the display canvas (expects a `canvas` named `display` and a `video` named `player` on `document`), plays audio, routes remote-control/gamepad input, and exposes the public API (`initialize`, `subscribe`, `execute`, `terminate`, `sendKeyPress`, `debug`, …). See `docs/engine-api.md`.
- **Worker library** — entry `src/core/index.ts`, output `brs.worker.js`. Runs in a **Web Worker** (browser) or **Worker Thread** (Node). Its `onmessage` handler receives a msgpack-encoded `AppPayload`/`TaskPayload` (load + run an app) or the `SharedArrayBuffer` for control state (`BrsDevice.setSharedArray`). This is where the interpreter actually executes.

The Node CLI runs the interpreter on a **single thread**; remote control there requires the ECP server (`--ecp`).

### Interpreter pipeline (`src/core/`)

`lex → parse → preprocess → interpret`

- `src/core/lexer/` — tokenizer.
- `src/core/parser/` — builds the AST (`Expression.ts`, `Statement.ts`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lvcabral/brs-engine](https://github.com/lvcabral/brs-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
