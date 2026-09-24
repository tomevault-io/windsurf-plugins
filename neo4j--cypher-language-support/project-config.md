---
trigger: always_on
description: This file provides guidance to coding agents (Claude Code, Codex, Cursor, …) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents (Claude Code, Codex, Cursor, …) when working with code in this repository.

## Repository overview

pnpm monorepo (workspaces: `packages/*`) implementing Neo4j's Cypher language support: syntax highlighting, autocompletion, linting, and formatting. The parser is generated with `antlr-ng` (no Java required) and runs on the `antlr4ng` runtime.

## Commands

```sh
pnpm install
pnpm build            # required before tests: generates the ANTLR parser, TextMate grammar, and bundles
pnpm test             # unit tests (vitest) across all packages
pnpm test:e2e         # e2e tests (run `pnpm exec playwright install` once first). Ask user to run these, they requires container manager and pop up windows on execution
pnpm lint             # oxlint (not eslint); pnpm lint-fix to autofix 
pnpm format           # oxfmt (not prettier); pnpm format:check to verify without autofix
pnpm build-vscode     # build only the VS Code extension and its dependencies
```

Run a single test file from the repo root (vitest workspace covers `language-support`, `lint-worker`, `react-codemirror`, `query-tools`):

```sh
pnpm vitest run packages/language-support/src/tests/lexer.test.ts
pnpm vitest run -t "test name substring"
```

Package-specific test suites:

- `pnpm --filter @neo4j-cypher/query-tools test:polling` — schema-polling integration tests (testcontainers, needs Docker).
- `pnpm --filter @neo4j-cypher/react-codemirror test:e2e` — Playwright component tests (`test:e2e-ui` for UI mode).
- `pnpm --filter neo4j-for-vscode test:apiAndUnit` — VS Code API/unit tests (mocha via `@vscode/test-electron`). Needs container manager like Docker/Rancher to run Neo4j
- `pnpm --filter neo4j-for-vscode test:webviews` — webview tests (wdio).
- `pnpm test:formattingIntegrity` — formatter verification over a large query corpus.

## Shell, paths and tooling (Windows dev machine)

- The Bash tool is Git Bash (POSIX) — PowerShell cmdlets (`Select-String`, `Select-Object`, …) do not exist there. In the PowerShell tool, never append `2>&1` to a native command: PowerShell 5.1 wraps stderr in `NativeCommandError` records and hides the real output (stderr is captured for you anyway).
- The shell working directory persists between calls (even across the Bash and PowerShell tools), so after any `cd` later relative paths silently resolve against the wrong directory. Relative paths work fine from the repo root — just `cd` back (or use absolute paths) rather than assuming you're still there.
- In Git Bash, unquoted backslash paths lose their backslashes (`C:UsersIsak...`) — quote them or use forward slashes.
- Repo tools are not on PATH: use `pnpm exec tsc`, `pnpm vitest run ...`, `pnpm format`. The test runner is vitest via pnpm — never jest, and this is a pnpm workspace, so `npm --workspace` does not work.

## Architecture

Dependency flow: `vscode-extension` bundles `language-server`, which (like `react-codemirror`) is powered by `language-support` and `lint-worker`; `query-tools` supplies the live database schema.

- **`packages/language-support`** — the core library. Everything else wraps it.
  - The parser is generated from the ANTLR4 grammar in `src/antlr-grammar/` into `src/generated-parser/` at build time (`pnpm gen-parser`). `Cypher25Lexer/Parser.g4` and `src/syntaxValidation/semanticAnalysis.js` (Neo4j's real semantic analysis, compiled to JS) originate in the internal Neo4j monorepo and are bumped monthly by an automated PR — do not hand-edit them. `CypherCmd*` and `CypherPre*` grammars are repo-local (e.g. the `RCURLY` override in `CypherCmdLexer.g4` embeds an antlr4ng-specific action).
  - Feature areas: `autocompletion/` (built on the `antlr4-c3` completion core), `syntaxValidation/` (lexer/parser errors + semantic analysis + schema-based validation), `formatting/` (see `src/formatting/overview.md`), `syntaxHighlighting/`, `signatureHelp.ts`.
  - Most features take a `DbSchema` (labels, procedures, databases, …) to give schema-aware results.
- **`packages/lint-worker`** — wraps linting in a `workerpool` worker so it runs off-thread. Built both as CJS (`lintWorker.cjs`, for the language server / VS Code) and ESM (`lintWorker.mjs`, for react-codemirror); consumers copy the bundle into their own dist at build time. Also pins an older published `@neo4j-cypher/language-support` for linting against older Neo4j versions (the extension's "Select Cypher linter version" command downloads other versions at runtime).
- **`packages/query-tools`** — Neo4j connection management and schema polling (`schemaPoller.ts`, `metadataPoller.ts`) that keeps the `DbSchema` up to date.
- **`packages/language-server`** — thin LSP wrapper; esbuild-bundles everything into a single `dist/cypher-language-server.js`.
- **`packages/vscode-extension`** — bundles the language server and lint worker into its `dist/`. Connection UI is React webviews under `src/webviews/`. Not published to npm (packaged with `vsce`).
- **`packages/react-codemirror`** (+ `react-codemirror-playground`) — CodeMirror 6 plugins and React wrapper, using `language-support` directly in the browser.

## Conventions and gotchas


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neo4j/cypher-language-support](https://github.com/neo4j/cypher-language-support) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
