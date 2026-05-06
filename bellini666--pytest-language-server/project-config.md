---
trigger: always_on
description: **IMPORTANT**: Never commit or push without explicit user confirmation. Always ask first.
---

# CLAUDE.md - AI Agent Guide

## Workflow Rules

**IMPORTANT**: Never commit or push without explicit user confirmation. Always ask first.

## Project Overview

**pytest-language-server** is a Rust LSP for pytest fixtures providing go-to-definition, find-references, hover, completions, diagnostics, and more.

- **Language**: Rust (Edition 2021, MSRV 1.85)
- **Framework**: `tower-lsp-server` + `rustpython-parser`
- **Run tests**: `cargo test`
- **Lint**: `cargo clippy`
- **Debug**: `RUST_LOG=debug cargo run`

## Architecture

```
src/
├── main.rs                 # LanguageServer trait impl + CLI entry point
├── lib.rs                  # Library exports
├── config/mod.rs           # Config from pyproject.toml [tool.pytest-language-server]
├── fixtures/               # Core analysis engine
│   ├── mod.rs              # FixtureDatabase struct (DashMap-based concurrent storage)
│   │                       #   + get_name_to_import_map() (cached, content-hash invalidated)
│   ├── types.rs            # FixtureDefinition, FixtureUsage, TypeImportSpec, etc.
│   ├── analyzer.rs         # Python AST parsing, fixture extraction, return-type import resolution
│   ├── import_analysis.rs  # Shared import layout analysis (AST + string fallback):
│   │                       #   ImportLayout, ImportGroup, ImportKind (Future/Stdlib/ThirdParty),
│   │                       #   parse_import_layout(), classify_import_statement(),
│   │                       #   adapt_type_for_consumer(), import_sort_key(), find_sorted_insert_position()
│   ├── imports.rs          # Import handling, is_stdlib_module(), build_name_to_import_map(), file_path_to_module_path()
│   ├── resolver.rs         # Fixture resolution with pytest priority rules
│   ├── scanner.rs          # Workspace + venv scanning
│   └── cli.rs              # CLI commands (fixtures list/unused)
└── providers/              # LSP handlers (one file per feature)
    ├── mod.rs              # Backend struct, URI/path helpers
    ├── code_action.rs      # Code actions: quickfix, source.pytest-ls, source.fixAll.pytest-ls
    │                       #   Uses import_analysis for layout + adapt; TextEdit production stays here
    ├── inlay_hint.rs       # Inlay hints with import-context-aware type display (adapt_type_for_consumer)
    ├── definition.rs, references.rs, hover.rs, completion.rs, ...
```

**Key pattern**: `FixtureDatabase` in `src/fixtures/` handles all data; `Backend` in `src/providers/` delegates LSP requests to it.

## Critical Knowledge

### Code Action Kinds
The code action provider (`src/providers/code_action.rs`) emits three kinds:

| Kind | Trigger | Behaviour |
|------|---------|-----------|
| `quickfix` | `undeclared-fixture` diagnostic | Adds missing fixture param with type annotation + import |
| `source.pytest-ls` | Cursor on unannotated fixture param | Adds `: ReturnType` + import for that fixture |
| `source.fixAll.pytest-ls` | Anywhere in file | Adds all missing type annotations + imports in one edit |

**Import insertion** is isort/ruff-aware:
- `parse_import_layout()` (in `import_analysis.rs`) parses the file via AST (or string fallback on
  syntax errors), returning an `ImportLayout` with classified `ImportGroup`s, `ParsedFromImport`s,
  and `ParsedBareImport`s.  `ImportKind` now has three variants: `Future`, `Stdlib`, `ThirdParty`.
- `emit_kind_import_edits()` inserts into the correct group with proper blank-line separators.
  Merging into **multiline** parenthesised imports is now supported (AST path only).
- `ImportLayout::find_matching_from_import()` finds existing `from X import Y` lines (single-line
  or multiline) for merge; `can_merge_into()` guards against merging fallback multiline entries
  whose names are unknown.
- `build_import_edits()` orchestrates deduplication, skip-if-already-imported, and group routing.

### TypeImportSpec & Return-Type Import Resolution
`TypeImportSpec` (in `types.rs`) captures `check_name` + `import_statement` for each type used in a fixture's return annotation. Resolved at analysis time:
1. `build_name_to_import_map()` (in `imports.rs`) builds a name→spec map from all imports in the fixture file (including stdlib/typing)
2. `resolve_return_type_imports()` (in `analyzer.rs`) tokenises the return type string, skips builtins, looks up each identifier in the import map, and falls back to locally-defined names via `file_path_to_module_path()`
3. Results are stored in `FixtureDefinition::return_type_imports` for use by code actions

`is_stdlib_module()` is a free function in `imports.rs`, used internally by `import_analysis.rs`
for classification.  It is no longer re-exported from `mod.rs` since all callers outside
`fixtures/` now go through `classify_import_statement()` in `import_analysis.rs`.

### Import-Aware Type Display (Inlay Hints)
`inlay_hint.rs` calls `adapt_type_for_consumer()` (from `import_analysis.rs`) before emitting each
hint, so the displayed type matches the consumer file's import style:
- If the consumer has `from pathlib import Path`, the hint shows `: Path` not `: pathlib.Path`.
- If the consumer has `import pathlib`, the hint shows `: pathlib.Path` not `: Path`.
The returned `Vec<TypeImportSpec>` is discarded — hints are display-only.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bellini666/pytest-language-server](https://github.com/bellini666/pytest-language-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
