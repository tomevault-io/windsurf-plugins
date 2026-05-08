---
trigger: always_on
description: **Ignition Dev Tools** is a multi-editor monorepo providing full IDE support for Inductive Automation's Ignition SCADA platform. It delivers LSP completions, hover docs, diagnostics, script decode/encode, project browsing, and more — for both **Neovim** and **VS Code**.
---

# CLAUDE.md — Ignition Dev Tools

## Project Overview

**Ignition Dev Tools** is a multi-editor monorepo providing full IDE support for Inductive Automation's Ignition SCADA platform. It delivers LSP completions, hover docs, diagnostics, script decode/encode, project browsing, and more — for both **Neovim** and **VS Code**.

Part of the **Whiskey House Ignition Developer Toolkit**:
- **Ignition Dev Tools** — Neovim + VS Code IDE support (this repo)
- **ignition-lint** — Static analysis for Ignition Python scripts
- **ignition-git-module** — Native Git inside Ignition Designer

## Monorepo Structure

```
packages/
├── ignition-lsp/        # Python LSP server (shared by both editors)
├── ignition-nvim/       # Neovim plugin (Lua)
└── ignition-vscode/     # VS Code extension (TypeScript)
```

Top-level symlinks (`lua/`, `lsp/`, `ftdetect/`, `ftplugin/`, `plugin/`, `syntax/`, `queries/`, `doc/`, `lazy.lua`) point into `packages/` so the repo works directly as a Neovim plugin via lazy.nvim's `runtimepath` discovery.

## Architecture

### Python LSP Server (`packages/ignition-lsp/`)
pygls 2.0 language server — shared by both editors.

| Module | Responsibility |
|--------|---------------|
| `server.py` | Main pygls server, document sync, request routing |
| `completion.py` | Context-aware completions for `system.*` and project scripts |
| `hover.py` | Hover documentation from API database |
| `diagnostics.py` | Integration with `ignition-lint` JythonValidator |
| `definition.py` | Go-to-definition for `system.*` and project scripts |
| `api_loader.py` | Loads and indexes API definitions from `api_db/` JSON |
| `project_scanner.py` | Walks Ignition project dirs, builds script index |
| `workspace_symbols.py` | Exposes project index via LSP workspace symbols |
| `api_db/*.json` | 14 modules, 239 functions — follows `api_db/schema.json` |
| `java_db/*.json` | Java/Jython class completions (26 packages, 146 classes) |
| `stubs/**/*.pyi` | Pyright/Pylance type stubs for Ignition APIs |

### Neovim Plugin (`packages/ignition-nvim/`)
Core Neovim plugin loaded at runtime.

| Module | Responsibility |
|--------|---------------|
| `lua/ignition/init.lua` | Entry point, `setup(opts)`, initializes subsystems |
| `lua/ignition/config.lua` | Configuration schema, validation, defaults |
| `lua/ignition/encoding.lua` | Encode/decode scripts (Ignition Flint format) |
| `lua/ignition/json_parser.lua` | Find embedded scripts in JSON by key names |
| `lua/ignition/decoder.lua` | Decode workflow: interactive selection, decode all, list scripts |
| `lua/ignition/virtual_doc.lua` | Virtual buffer system: `acwrite` buffers, auto-save, source tracking |
| `lua/ignition/lsp.lua` | LSP client via `vim.lsp.start()` (Neovim 0.11+) |
| `lua/ignition/kindling.lua` | Integration with Kindling for `.gwbk` files |

### VS Code Extension (`packages/ignition-vscode/`)
TypeScript extension providing IDE features in VS Code.

| Component | Responsibility |
|-----------|---------------|
| `src/extension.ts` | Extension entry point, activation, command registration |
| `src/lsp/` | LSP client management, auto-install of ignition-lsp |
| `src/encoding/` | Decode/encode scripts, CodeLens for embedded scripts |
| `src/views/projectBrowser.ts` | Designer-style project tree sidebar |
| `src/views/tagBrowser.ts` | Tag browser for ignition-git-module tag exports |
| `src/views/componentTree.ts` | Perspective view component hierarchy |
| `resources/` | Sidebar icons and static assets |
| `syntaxes/` | TextMate grammars for syntax highlighting |
| `test/` | Vitest test suite |

### Supporting Files
- `lazy.lua` → symlink to `packages/ignition-nvim/lazy.lua` (lazy.nvim plugin spec)
- `ftdetect/`, `ftplugin/`, `syntax/`, `plugin/`, `queries/`, `doc/` → symlinks for Neovim runtimepath
- `lsp/` → symlink to `packages/ignition-lsp/`
- `website/` — Docusaurus documentation site
- `claude-code-plugin/` — Claude Code plugin for Ignition awareness
- `templates/` — Per-project Claude Code templates

## Domain Context

**Ignition** is a SCADA/ICS platform by Inductive Automation. Projects are stored as JSON files containing embedded Python (Jython) scripts. Developers use `system.*` scripting APIs (e.g., `system.tag.readBlocking()`, `system.db.runPrepQuery()`, `system.perspective.sendMessage()`).

Both editors decode those embedded scripts into editable Python buffers with full LSP support, then encode them back.

## Critical Technical Details

### Encoding (HANDLE WITH EXTREME CARE)
- Uses **Ignition Flint format** — plain string replacement, NOT base64
- Standard JSON escapes: `\"`, `\n`, `\t`, `\b`, `\r`, `\f`
- Unicode escapes: `<` → `\u003c`, `>` → `\u003e`, `&` → `\u0026`, `=` → `\u003d`, `'` → `\u0027`
- **Order matters:** backslash first to avoid double-escaping
- **Round-trip fidelity is sacred:** `encode(decode(x)) == x` must always hold
- Lua encoding uses `string.gsub` with **plain flag** — NOT Lua patterns (this caused bugs)
- VS Code encoding in `src/encoding/` mirrors the same logic in TypeScript

### LSP Client

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheThoughtagen/ignition-ide-plugins](https://github.com/TheThoughtagen/ignition-ide-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
