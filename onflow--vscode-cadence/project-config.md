---
trigger: always_on
description: handles detection, version selection, and the `cadence.changeFlowCliBinary` command.
---

# AGENTS.md

Guidance for AI coding agents (Claude Code, Codex, Cursor, Copilot, and others) working in
this repository. Loaded automatically into agent context — keep edits concise.

## Overview

This repo is the official VS Code extension for Cadence, the resource-oriented smart contract
language of the Flow network. It is a TypeScript extension (`publisher: onflow`, extension id
`cadence`, version per `package.json`) bundled with esbuild, ships a TextMate grammar, a
language-client that talks to the Cadence language server embedded in the Flow CLI, a debug
adapter registration (`type: cadence`), a test provider for `.cdc` test files, and a JSON
schema provider that serves a `flow.json` schema via a virtual `cadence-schema://` filesystem.

Targets `vscode ^1.99.3` (`package.json` `engines.vscode`). CI builds on Node 22.x
(`.github/workflows/ci.yml`).

## Build and Test Commands

Install: `npm install` (CI uses `npm ci`).

Build (esbuild bundles `extension/src/main.ts` to `out/extension/src/main.js` and copies
`cadence-parser.wasm` from `@onflow/cadence-parser` into `out/extension/`):

- `npm run esbuild` — one-shot dev build with sourcemaps
- `npm run esbuild-watch` — rebuild on change
- `npm run check` — `tsc -p ./` type-check only (no emit beyond `tsconfig.json` outDir)
- `npm run package` — produces a `.vsix` via `vsce package`
- `npm run install-extension` — `code --install-extension cadence-*.vsix` (depends on prior `package`)
- `npm run package-test` — packages to `./extension/test/fixtures/workspace/cadence.vsix`

Lint (`ts-standard`, `tm-tests/**` is ignored per `package.json` `ts-standard.ignore`):

- `npm run lint`
- `npm run lint-fix`

Tests:

- `npm test` — full integration suite: `clean-test` → `esbuild` → `tsc -p ./` →
  `copy-test-fixtures` → `node ./out/extension/test/run-tests.js`. Runner uses
  `@vscode/test-electron` to download VS Code and launch against
  `./extension/test/fixtures/workspace` (`extension/test/run-tests.ts`). On Linux CI, Xvfb
  is started before this step.
- `npm run test-grammar` — mocha against `tm-tests/grammar.test.mjs`
- `npm run test-grammar-regression` — mocha against `tm-tests/regression.test.mjs`
- `npm run test-grammar-all` — both grammar suites

Dev inner loop (per `CONTRIBUTING.md`): run `tsc -watch -p ./`, then press F5 in VS Code to
launch an Extension Host. Reload the host after TS changes.

## Architecture

Entry point is `extension/src/main.ts` (declared in `package.json` `main` as
`./out/extension/src/main.js`). `extension/src/extension.ts` is the top-level class wired from
`main.ts`. Activation events: `onLanguage:cadence` and `onFileSystem:cadence-schema`
(`package.json` `activationEvents`).

`extension/src/` layout:

- `server/language-server.ts` — spawns and manages the Cadence language server embedded in
  the Flow CLI; uses `vscode-languageclient`.
- `flow-cli/` — `cli-provider.ts`, `cli-selection-provider.ts`, `cli-versions-provider.ts`;
  handles detection, version selection, and the `cadence.changeFlowCliBinary` command.
- `dependency-installer/` — `dependency-installer.ts` + `installers/flow-cli-installer.ts`,
  `installers/homebrew-installer.ts`; drives the `cadence.checkDepencencies` command
  (note: typo is intentional — it matches `command-constants.ts` and `package.json`
  `contributes.commands`).
- `commands/` — `command-controller.ts` registers the three contributed commands
  (`cadence.restartServer`, `cadence.checkDepencencies`, `cadence.changeFlowCliBinary`).
- `test-provider/` — VS Code TestController for `.cdc` test files; `test-runner.ts`,
  `test-resolver.ts`, `test-trie.ts`. Concurrency is bounded by `cadence.test.maxConcurrency`.
- `json-schema-provider.ts` — implements the `cadence-schema://` FileSystemProvider that
  backs `contributes.jsonValidation` for `flow.json` (see `flow-schema.json` at repo root).
- `settings/settings.ts` — reads contributed configuration (`cadence.flowCommand`,
  `cadence.accessCheckMode`, `cadence.customConfigPath`, `cadence.test.maxConcurrency`).
- `telemetry/` — Sentry (`sentry-wrapper.ts`) + Mixpanel (`mixpanel-wrapper.ts`).
- `storage/`, `ui/`, `utils/` — state caching, prompts/notifications, shell exec helpers.
- `crypto-polyfill.ts` — Node crypto shim required by bundled deps.

`extension/language/` — `language-configuration.json`, `syntaxes/cadence.tmGrammar.json`
(TextMate grammar registered under `contributes.grammars`), and
`syntaxes/codeblock.json` (injected into `text.html.markdown` for fenced Cadence blocks).

`extension/test/` — `run-tests.ts` (VS Code test launcher), `index.ts`, `globals.ts`,
`integration/` (numbered `0–6` test files: dependencies, language-server, commands, schema,
test-trie, test-provider), `unit/` (`state-cache.test.ts`, `test-trie.test.ts`),
`fixtures/workspace/` (Cadence files + nested `flow.json`s used by integration tests),
`mock/`.

`tm-tests/` — TextMate grammar tests (`grammar.test.mjs`, `regression.test.mjs`) run with
mocha. CI only runs these when `tm-tests/**` or the `cadence.tmGrammar.json` changes
(paths-filter in `.github/workflows/ci.yml`).

`scripts/` — `extract-contract.mjs`, `print-scopes.mjs` (grammar tooling).

`docker-compose.yml` runs `codercom/code-server` against the test fixtures workspace on

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [onflow/vscode-cadence](https://github.com/onflow/vscode-cadence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
