---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo layout

This is a **dual-stack monorepo**: a Python LSP server and a TypeScript VS Code extension that talk to each other.

- `src/marimo_lsp/` — Python LSP server (pygls-based). Bridges VS Code's notebook protocol to a marimo `Session`/kernel.
- `extension/` — VS Code extension (TypeScript, Effect-TS). Consumes the LSP, renders cells, manages kernels.
- `tests/` — pytest tests for the LSP server (uses `pytest-lsp` for in-process client).
- `extension/src/**/__tests__/` — vitest unit tests, colocated with source.
- `extension/tests/` — `@vscode/test-cli` integration tests that launch a real VS Code instance.
- `scripts/` — release / maintenance Python scripts (e.g. `generate_release_notes.py`).

The Python server is pinned to a specific `marimo` version (`.marimo-version`). For local dev, clone `marimo-team/marimo` as a sibling directory and check out that tag — the extension links `@marimo-team/frontend`, `@marimo-team/openapi`, and `@marimo-team/smart-cells` from `../../marimo/`. CI does this automatically.

## Common commands

Toolchain: **`uv`** for Python, **`pnpm`** for Node, **`just`** as task runner.

**NEVER** invoke `python`/`pip` directly — use `uv run ...` / `uv pip ...`.

Run `just --list` to see available recipes. They're bucketed into `lint`, `fix`, `test`, `build`, and `setup` groups; recipes that accept pytest/vitest args take them as trailing positional args (`just test-py -v -k name`, `just test-ts --watch`).

To launch the extension for manual testing, open the repo in VS Code and press **F5**.

## Architecture

Read `ARCHITECTURE.md` first — it documents the custom LSP protocol (commands like `marimo.run`, `marimo.serialize`, `marimo.set_ui_element_value`, and the `marimo/operation` server→client notification stream). The key mental model:

1. **VS Code notebook protocol** handles cell text sync (`notebookDocument/didOpen`, `didChange`, ...).
2. **Custom LSP commands** drive kernel actions (run cells, interrupt, call UI element functions, (de)serialize).
3. **`marimo/operation` notifications** stream kernel output/state (`cell-op`, `variables`, `data-*-preview`, `alert`, package install progress, …) from server to client.

### Python side (`src/marimo_lsp/`)

- `server.py` — pygls `LanguageServer` with handlers for notebook lifecycle + custom commands.
- `session_manager.py` — `LspSessionManager`: notebook URI → marimo `Session`. Sessions are created lazily on first `marimo.run` and closed on untitled-close, executable change, or shutdown. **URIs are the session key** — they're unstable across rename, so some operations can lose track.
- `session.py`, `session_consumer.py`, `app_file_manager.py`, `kernel_manager.py` — LSP-flavored adapters around marimo's internals (`SessionConsumer`, `AppFileManager`, `KernelManager`) that read from in-memory LSP documents instead of files.
- `completions.py`, `diagnostics.py`, `_rules.py` — language features (trigger-char completion for `@cell_name`, diagnostics).
- `package_manager.py`, `api.py`, `models.py` — package-install flow and shared LSP request/response schemas.

### TypeScript side (`extension/src/`)

Built on **Effect-TS**. Everything is wired as `Layer`s (services) assembled in `layers/Main.ts` via `makeActivate(...)` in `features/Main.ts`. Directory roles:

- `features/` — top-level layers that only register side effects (commands, codelens, file detection, theme sync, debug, cell-metadata bindings).
- `services/`, `kernel/`, `notebook/`, `config/`, `lsp/`, `platform/`, `python/`, `telemetry/` — the actual services composed into `MainLive`. E.g. `KernelManager` consumes `marimo/operation`, `ExecutionRegistry` drives `NotebookCellExecution` state, `CellStateManager` tracks stale cells and the `marimo.notebook.hasStaleCells` context key, `NotebookRenderer` renders `application/vnd.marimo+html` MIME outputs, `VariablesService`/`DatasourcesService`/`PackagesService` back the tree views.
- `commands/` — one file per VS Code command registered in `package.json`.
- `renderer/` — webview-side notebook renderer (`renderer.tsx`) that embeds marimo's frontend.
- `panel/`, `views/` — tree views for recents, variables, datasources, packages.
- `lsp/` — language-client wiring + inner servers (`RuffLanguageServer`, `TyLanguageServer`) used when managed language features are enabled.
- `lib/`, `utils/`, `schemas/`, `vendor/`, `__mocks__/` — supporting code.

Cells use a dedicated language id `mo-python` so external Python LSPs don't double up on completions/diagnostics. Users can opt out with `marimo.disableManagedLanguageFeatures`.

## Philosophy

Tests, types, and lints are pedantic on purpose: they're load-bearing tools, not ceremony. Opting out is rarely the right call, but is allowed only with a thoughtful and clear explanation of *why*.

## Effect-TS


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marimo-team/marimo-lsp](https://github.com/marimo-team/marimo-lsp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
