---
trigger: always_on
description: A Language Server Protocol implementation for Lua (World of Warcraft API dialect). Provides hover, go-to-definition, completion, signature help, find references, rename, and diagnostics.
---

# wowlua_ls — WoW Lua Language Server

A Language Server Protocol implementation for Lua (World of Warcraft API dialect). Provides hover, go-to-definition, completion, signature help, find references, rename, and diagnostics.

For deep architecture internals, see [ARCHITECTURE.md](.claude/ARCHITECTURE.md) — type inference, narrowing, generics, builder pattern, cross-file references, metatable inference, flavor filtering, plus the relocated source-file deep notes, config internals, inlay hints, the per-feature annotation/type-system reference, and the diagnostics module reference. Pointers below lead to the relevant section.

For the full test-file catalog and the embedded-assertion (`hover:`/`def:`/`diag:`/…) format, see [TESTING.md](.claude/TESTING.md).

For Neovim diagnostic integration details (push/pull namespaces, `workspace_diagnostics` flag, line-shifting, edit zone handling), see [NEOVIM_DIAGNOSTICS.md](.claude/NEOVIM_DIAGNOSTICS.md).

## Architecture

### Workspace crates
The project is a cargo workspace of layered library crates plus a thin binary. The layering is **one-directional and enforced at compile time** — a crate can only use the crates below it:

- **`wowlua_syntax`** (`crates/wowlua_syntax/`) — leaf crate: lexer, parser, CST (`syntax/`), typed AST (`ast.rs`). Depends on nothing else.
- **`wowlua_core`** (`crates/wowlua_core/`) — the shared type vocabulary: IR types (`types.rs`), flavor bitmask (`flavor.rs`), and the annotation type *definitions* embedded in the IR (`annotations.rs`: `AnnotationType`, `TuplePosition`, `ParamInfo`, `Visibility`, `KEYOF_SELF_TARGET`). Re-exports `syntax`/`ast`.
- **`wowlua_analysis`** (`crates/wowlua_analysis/`) — the per-file analysis engine and everything in its dependency cycle: `analysis/`, `annotations/` (parsing/scanning; re-exports the core type defs), `pre_globals/`, `diagnostics/`, `config.rs`, `xml_scan.rs`. Owns `MAX_COMPLETIONS`. A `test-util` feature exposes `#[cfg(test)]` construction helpers (`ClassDecl`/`ExternalGlobal::for_test`, `PreResolvedGlobals::push_ext_*`) to higher crates' test builds.
- **`wowlua_lsp`** (`crates/wowlua_lsp/`) — `lsp/` (server loop + handlers), `plugins/` (Lua plugin engine), `toc/` (.toc parsing), `has_shebang`. Owns the `embedded-stubs` feature (`lsp/main_loop/stub_loading.rs`).
- **`wowlua_stub_gen`** (`crates/wowlua_stub_gen/`) — the offline stub-generation tool (`stub_gen/`); above `wowlua_lsp` because it drives a workspace scan. Forwards `embedded-stubs` to `wowlua_lsp`.
- **`wowlua_doc`** (`crates/wowlua_doc/`) — Markdown doc generation (`doc_gen.rs`, `doc_gen_md.rs`); depends only on `wowlua_analysis`, parallel to `wowlua_lsp`.
- **`wowlua_ls`** (root `src/`) — thin binary: `main.rs` + `cli/`, plus a facade `lib.rs` that re-exports every lower crate's modules so `wowlua_ls::<module>` (tests, CLI) and intra-crate `crate::<module>` paths resolve unchanged.

**Conventions for working across the split:**
- Each crate's `lib.rs` re-exports the modules of the crates below it (`pub use wowlua_core::{...}`), so the original `crate::syntax::…`/`crate::types::…`/etc. paths inside moved code resolve without edits. When a type/const must move *down* a layer, re-export it from its original module path so existing `crate::<old_path>::Name` references keep working (e.g. `annotations/mod.rs` re-exports the core annotation type defs).
- Items that cross a crate boundary must be `pub`, not `pub(crate)` — within each split-out crate, `pub(crate)` was promoted to `pub`. **Module-level privacy is preserved**: the `Ir`/`PreResolvedGlobals` arena fields and the `lsp` `main_loop` struct fields are plain `private` (never `pub(crate)`), so the promotion didn't touch them and they stay encapsulated behind their routing surfaces.
- The detailed module descriptions below use pre-split `src/<module>` paths; those files now live under `crates/<crate>/src/<module>` per the mapping above (only `main.rs`, `cli/`, and the facade `lib.rs` remain in the root `src/`).
- `stubs/` stays at the workspace root. Code that locates it via `env!("CARGO_MANIFEST_DIR")` (the `include_bytes!` in `stub_loading.rs`, the regenerate-stubs output path) uses `../../stubs` to climb from `crates/<crate>/` to the root.

### Source files
A concise map of the source tree. Several files carry deep mechanism notes too large for an always-loaded map; those are relocated to [ARCHITECTURE.md — Source-file deep notes](.claude/ARCHITECTURE.md#source-file-deep-notes) with a pointer left on the relevant bullet.

- `src/main.rs` — CLI entry point: `evaluate` subcommand, `test-query` subcommand (hover/def/sig/completions/diagnostics), `dump-types` subcommand (hover regression baselines), `doc` subcommand (markdown API doc generation), otherwise starts LSP
- `src/doc_gen.rs` — Documentation data model: `DocNamespace`, `DocDefine`, `DocField`, `DocParam` structs, standalone type formatter operating on `PreResolvedGlobals`, class/field iteration with visibility and source locations
- `src/doc_gen_md.rs` — Markdown documentation renderer: takes `Vec<DocNamespace>` and produces VitePress-compatible `.md` files (one per class + `index.md`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TradeSkillMaster/wowlua-ls](https://github.com/TradeSkillMaster/wowlua-ls) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
