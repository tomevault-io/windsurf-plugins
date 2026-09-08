---
trigger: always_on
description: This file is a concise guide for AI coding agents (and human contributors) working on this repository. It describes what the project does, how the code is organized, which commands to run, and the important conventions and pitfalls to respect.
---

# Agent Guide — @tomjs/vite-plugin-vscode

This file is a concise guide for AI coding agents (and human contributors) working on this repository. It describes what the project does, how the code is organized, which commands to run, and the important conventions and pitfalls to respect.

## Project Overview

[@tomjs/vite-plugin-vscode](https://github.com/tomjs/vite-plugin-vscode) is a [Vite](https://vite.dev/) plugin for developing [VS Code extension webviews](https://code.visualstudio.com/api/references/vscode-api#WebviewPanel) with `vue`/`react` (or any Vite-supported framework). It wires the webview renderer build into the VS Code extension build:

- **Compiles the consumer's `extension` code** with Vite itself (Rolldown-based, since Vite 8) — replacing the old tsdown/tsup pipeline — with `vscode` and Node.js built-ins kept external.
- **Injects the webview renderer** into the extension at runtime: in dev it injects the `client.iife.js` script into the served HTML for `HMR`; in production it injects the final generated `index.html` (with CSP/nonce/baseUri rewriting) into the extension code via the `virtual:vscode` module.
- Supports **`esm` and `cjs`** extension output, multi-page webviews (`rollupOptions.input`), `vue`/`react` devtools injection, and an optional **electron-builder-free** packaging story.
- Ships three distributable artifacts from `src/`: the plugin itself (`index.js`), a small `getWebviewHtml` helper (`webview.js`), and the webview client shim (`client.iife.js`).

The package is published on npm as `@tomjs/vite-plugin-vscode`.

## Tech Stack

- **Language**: TypeScript (strict, `@tomjs/tsconfig` base), ESM (`"type": "module"`).
- **Build tool for this package**: [tsdown](https://tsdown.dev/) (see `tsdown.config.ts`) — **the library itself is built with tsdown**; only the _consumer's_ `extension` code compilation moved to Vite.
- **Vite**: peer `^8.0.0` (Vite 8 is Rolldown-based). The plugin invokes Vite's `build()` API to compile the consumer's extension.
- **Package manager**: [pnpm](https://pnpm.io/) (`packageManager: pnpm@10.26.2`, workspace root, `examples/*`).
- **Linting**: ESLint via `@tomjs/eslint-config` (flat config), stylelint via `@tomjs/stylelint-config` for example stylesheets.
- **Commit conventions**: `commitlint` with `@tomjs/commitlint-config`, enforced by `simple-git-hooks` + `lint-staged`. Conventional Commits style (see commit history: `feat:`, `fix:`, `chore:`, `docs:`).
- **Runtime**: Node >= 18.19; peer dependencies `@types/vscode ^1.56.0`, `vite ^8.0.0`.
- **Runtime dependencies** (externalized in the library build): `@tomjs/node`, `execa`, `lodash.merge`, `node-html-parser`, `picocolors`.

## Repository Layout

```
|-- src/                  # Plugin source (the package itself)
|  |-- index.ts           # Plugin entry: option merging + all Vite plugin hooks (serve/build)
|  |-- build.ts           # Vite build orchestration for the consumer's extension (dev watch + prod)
|  |-- types.ts           # Public option interfaces (PluginOptions, ExtensionOptions, WebviewOption)
|  |-- constants.ts       # PLUGIN_NAME, ORG_NAME, VIRTUAL_MODULE_ID, RESOLVED_VIRTUAL_MODULE_ID
|  |-- logger.ts          # Vite logger wrapper with [tomjs:vscode] prefix
|  |-- utils.ts           # Dev server URL resolution helpers
|  |-- webview/
|  |  |-- webview.ts      # getWebviewHtml helper -> dist/webview.js (+ d.ts)
|  |  |-- client.ts       # webview client shim (acquireVsCodeApi patch, commands) -> dist/client.iife.js
|  |  |-- template.html   # dev HMR webview template (iframe to VITE_DEV_SERVER_URL)
|  |  |-- global.d.ts     # ambient module declarations ('*.html')
|  |  |-- window.d.ts     # Window.acquireVsCodeApi global
|-- examples/             # Runnable demo apps: react, vue, vue-esm (ESM ext), vue-import (multi-page)
|-- env.d.ts              # Public ambient types shipped to consumers (VITE_* env vars, __getWebviewHtml__)
|-- env-webview.d.ts      # Public 'virtual:vscode' module declaration
|-- tsdown.config.ts      # Build config for publishing this package (3 targets + d.ts)
|-- tsconfig.json         # TS project config (node) + references tsconfig.web.json
|-- tsconfig.web.json     # TS config for the browser-side webview client
|-- eslint.config.mjs / commitlint.config.mjs / stylelint.config.mjs
```

## Common Commands

Run from the repository root unless noted:

| Command            | Purpose                                                                |
| ------------------ | ---------------------------------------------------------------------- |
| `pnpm install`     | Install all dependencies (workspace root + examples).                  |
| `pnpm dev`         | Watch-build the package with tsdown (`pnpm clean && tsdown --watch`).  |
| `pnpm build`       | Build the package to `dist/` (tsdown). Also runs via `prepublishOnly`. |
| `pnpm clean`       | Remove `dist/`.                                                        |
| `pnpm lint`        | Run `stylelint` then `eslint --fix` over the repo.                     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomjs/vite-plugin-vscode](https://github.com/tomjs/vite-plugin-vscode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
