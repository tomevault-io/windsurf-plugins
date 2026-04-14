---
trigger: always_on
description: This file is the entrypoint for LLM contributors. It is a table of contents for high-value project context in `agents/`.
---

# AGENTS: Ousia Contributor Intelligence

This file is the entrypoint for LLM contributors. It is a table of contents for high-value project context in `agents/`.

## Non-Optional Maintenance Rule

If code, commands, architecture, semantics, tests, or workflows change, you must update this file and affected files under `agents/` in the same change set.

Do this autonomously. Do not defer it to “later docs cleanup”.

## How To Use This Pack

1. Read `agents/01-repo-map.md` for structure and boundaries.
2. Read `agents/02-compiler-pipeline.md` before touching parser/IR/codegen.
3. Read `agents/03-language-semantics.md` before changing language behavior.
4. Read `agents/04-testing-ci.md` before refactors, bugfixes, or snapshots.
5. Read `agents/05-engineering-playbook.md` before implementing non-trivial changes.

## Index

- `agents/01-repo-map.md`: repository map, ownership zones, and source-of-truth files.
- `agents/02-compiler-pipeline.md`: front-end to backend flow (`tokenizer` -> `parser` -> `ir` -> QBE verification -> runtime backend -> binary).
- `agents/03-language-semantics.md`: language model and invariants implemented today.
- `agents/04-testing-ci.md`: how to run tests, snapshot behavior, CI expectations, and debugging flow.
- `agents/05-engineering-playbook.md`: practical implementation rules for safe compiler evolution.

## Scope Note

This repository contains the Ousia compiler workspace (`crates/*`) plus editor tooling under `tools/vscode-ousia/`. Prefer touching compiler code only unless explicitly requested.

## API Stability Policy (Pre-Release)

- Ousia is pre-release; source compatibility and stdlib API stability are not guaranteed yet.
- Prefer semantic clarity, correctness, and maintainability over preserving legacy interfaces.
- If a clean fix requires an API break (language surface, stdlib helpers, CLI behavior, or internal crate APIs), take the break instead of adding compatibility shims.
- When introducing a breaking change, update tests/snapshots and all affected `agents/*` documentation in the same change set.

## Testing Runner Preference

- For Rust test execution, use `cargo nextest run` when `cargo-nextest` is available in the environment (it is the preferred default because it is faster).
- Fall back to `cargo test` only when `cargo-nextest` is unavailable or when parity with CI behavior must be verified explicitly.
- Tracked Git hooks live under `.githooks/`; enable them locally with `git config core.hooksPath .githooks`.
- The local `pre-commit` hook formats staged Rust files with nightly `rustfmt` (using `rustfmt.toml` with nightly import-sorting options), and the local `pre-push` hook is intentionally a no-op (no automatic test execution).
- Repo root `install-vscode-extension.sh` is the canonical helper for packaging `tools/vscode-ousia` into a `.vsix` and installing it through the VS Code CLI (`CODE_BIN` can override the `code` executable).

## Current Syntax Notes

- Generics use square brackets for type parameters and specialization arguments: `generic Option[T] { ... }`, `specialize OptionI32 = Option[I32]`.
- Generic bodies also support local specialization aliases (`specialize LocalAlias = Name[TypeArgs...]`) so generic helpers can reuse other generics after specialization.
- Traits are declaration-only method signature blocks: `trait Hash { fun hash(v: Self) -> I32 }`.
- Trait implementations are explicit and concrete-only in v1: `impl Hash for I32 { fun hash(v: I32) -> I32 { ... } }`.
- Generic bounds are inline on parameters: `generic HashTable[K: Hash + Equality, V] { ... }`.
- Legacy template syntax is hard-removed: `template` and `instantiate` are parser errors with migration hints to `generic` / `specialize`.
- Trait invocation in v1 is namespaced/static (`Trait.method(value, ...)`); calls are rewritten to concrete impl symbols during specialization and remain static dispatch only.
- Infix `+`, `-`, `*`, `/`, `==`, `!=`, `<`, `<=`, `>`, `>=` remain parser-level operators, but resolve routes non-primitive cases through operator traits (`Addition`, `Subtraction`, `Multiplication`, `Division`, `Equality`, `Comparison`) before backend lowering.
- Top-level imports are flat and same-directory only: `import "helpers.oa"`. Imported declarations are merged into one global scope.
- Same-directory import validation now lives in `crates/oac/src/flat_imports.rs` as shared helper `validate_same_dir_oa_import(...)` and is reused by both compiler import resolution and LSP project symbol/import traversal.
- Tokenization/parsing now also carry non-serialized provenance side data for verification artifacts: tokens track exact source spans plus optional source filenames, parser functions/invariants retain source-span maps keyed by shared AST-path vocabulary (`stmt:<n>/...`, `pre:<n>`), and flat-import loading preserves original imported file paths in those spans so emitted SMT comments can point back to the real source file.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ousialang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
