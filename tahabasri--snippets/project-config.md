---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A VS Code extension ("Snippets", publisher `tahabasri`) for managing code snippets and AI prompts in a tree view. It also runs in Cursor and Windsurf, which are detected at runtime and change available commands and welcome text.

## Commands

```bash
npm run compile        # webpack dev build -> dist/extension.js
npm run watch          # webpack watch mode for development
npm run package        # production webpack build (used by vscode:prepublish)
npm run lint           # eslint over src (--ext ts)
npm run test-compile   # tsc -p ./  -> compiles to out/ (required before tests)
npm test               # runs out/test/runTest.js via @vscode/test-electron
```

Run/debug the extension itself with VS Code's "Run Extension" launch config (F5), not from the CLI.

### Tests

- `npm test` launches a headless VS Code instance and runs every `out/**/*.test.js` through Mocha (TDD `suite`/`test` style, see `src/test/suite/index.ts`).
- Tests **must** be compiled first with `npm run test-compile` — `npm test` runs the JS in `out/`, not the TS in `src/`. `pretest` chains compile + lint automatically when you run `npm test`.
- There is no built-in single-test filter in the npm scripts. To run one suite, temporarily narrow the glob in `src/test/suite/index.ts` or use Mocha's `.only` on a `suite`/`test`, then recompile.
- Tests use `sinon` for stubbing the `vscode` API. Snippet-logic suites (`snippetService.test.ts`, `snippetProvider*.test.ts`) exercise the in-memory model; `aiIntegration.test.ts` covers the Copilot/Cursor/Gemini commands.

> Two build pipelines coexist: **webpack** produces the shipped bundle (`dist/`), **tsc** produces `out/` for tests only. `main` in package.json points at `dist/extension`.

## Architecture

The data flows in one direction: a `DataAccess` storage backend ⟶ `SnippetService` (in-memory tree + mutations) ⟶ `SnippetsProvider` (VS Code `TreeDataProvider`) ⟶ command functions in `config/commands.ts`. Everything is wired together in `activate()` in `src/extension.ts`.

### Snippet model (`src/interface/snippet.ts`)

Everything — folders and snippets — is a `Snippet`. A folder has `folder: true` and a `children` array; a leaf snippet has a `value`. The whole dataset is a single tree under one root node (`id: 1`, `parentId: -1`, see `DataAccessConsts.defaultRootElement`). IDs are integers handed out via a monotonic `lastId` stored on the root; `SnippetService.incrementLastId()` / `fixLastId()` keep it consistent (corruption here is the root cause of the "Troubleshoot Snippets" feature).

### Storage layer (`src/data/`)

`DataAccess` is an interface with two implementations — pick is by scope:
- `MementoDataAccess` — **global** snippets, persisted in `context.globalState` under key `snippetsData`. Registered for VS Code Settings Sync (`setKeysForSync`), so global snippets sync across machines.
- `FileDataAccess` — **workspace** snippets, persisted to `.vscode/snippets.json`. Enabled only when the `snippets.useWorkspaceFolder` setting is on. Also used transiently for import/export to arbitrary JSON files.

The whole tree is serialized/deserialized as one JSON blob on every save — there is no per-snippet persistence.

### Service (`src/service/snippetService.ts`)

Holds the root `Snippet` in memory and is the only place that mutates the tree (add/update/remove/move/sort/import/export). Recursive helpers `findParent`, `flatten`, and `flattenAndKeepFolders` are the workhorses. Call `service.saveSnippets()` to flush to the backing `DataAccess`; the service does **not** auto-save on mutation — `SnippetsProvider.sync()` is what persists + refreshes the tree.

### Provider (`src/provider/snippetsProvider.ts`)

Implements both `TreeDataProvider<Snippet>` and `TreeDragAndDropController<Snippet>`. Wraps `SnippetService`, converts `Snippet`s to `TreeItem`s (icons, context values, language binding), and after any mutation calls `sync()` → save + `refresh()` (fires `onDidChangeTreeData`). Drag-and-drop reparenting and the guard against dropping a folder into its own descendant live in `handleDrop`.

### Global vs Workspace duality

There are **two parallel instances** of the service+provider: a global pair (always present) and a workspace pair (`ws*`, created only when `useWorkspaceFolder` is enabled). This is why commands come in `globalSnippetsCmd.*` / `wsSnippetsCmd.*` variants plus shared `commonSnippetsCmd.*` ones that prompt the user to choose a target view. Two tree views are contributed: `snippetsExplorer` and `wsSnippetsExplorer`.

### Commands (`src/config/commands.ts`)

`CommandsConsts` enum is the single source of truth for command IDs and **must stay in sync with the `contributes.commands` / `menus` declarations in package.json**. The exported functions are the command bodies; they are registered in `activate()`. `when` clauses in package.json gate menu visibility on runtime context keys set via `setContext`: `snippets.host` (`vscode`|`cursor`|`windsurf`), `snippets.workspaceState` (`fileAvailable`), and `snippets.actionMode` (`button`|`inline`).

### IntelliSense completion


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tahabasri/snippets](https://github.com/tahabasri/snippets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
