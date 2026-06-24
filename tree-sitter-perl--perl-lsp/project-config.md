---
trigger: always_on
description: A Perl LSP server built on ts-parser-perl (crates.io) and tower-lsp.
---

# perl-lsp

A Perl LSP server built on ts-parser-perl (crates.io) and tower-lsp.

## Build & test

```
cargo build --release
cargo test                              # unit tests
./e2e/run.sh                            # e2e tests (needs nvim + release build)
gold-corpus/run.pl                      # gold harness (needs release build + installed substrate)
perl-lsp --dump-package <root> <pkg>    # debug type inference for a package
```

The gold harness (`gold-corpus/README.md`) is an exact-assertion regression
net over real CPAN modules from a snapshot-pinned substrate
(`gold-corpus/local/`). `fixtures/*.json` is the source of truth; statuses:
gold (must hold → FAIL on regression), xfail (known gap → XPASS when a fix
lands, promote the row), provisional (reported, never fails). A crash is
always a hard fail. Run it alongside `cargo test` + `./e2e/run.sh` before
calling a change verified; `gold-corpus/run.pl --emit <cap> <file> <row>
<col>` authors new rows. Known gaps live in `gold-corpus/KNOWN-GAPS.md`.

## Architecture

Four layers, data flows down only — enforced by `src/layering_tests.rs`
(the import DAG, the model's Point-only tree-sitter surface, and
single-point grammar access all fail `cargo test` on violation; new
source files must be assigned a layer in its `layer_map`):

```
LSP adapter      symbols.rs, backend.rs       → LSP protocol types
Cross-file       module_index/_resolver/_cache → CachedModule (Arc<FileAnalysis>)
Builder          builder.rs                   → produces FileAnalysis
Data model       file_analysis.rs             → FileAnalysis (serde, bincode-cacheable)
```

See `docs/ROADMAP.md` for the forward design corpus entry point. `docs/adr/file-store-and-resolve.md` covers the landed cross-file unification: single role-tagged FileStore + RoleMask + `refs_to`, plus `resolve_symbol` (cursor→`ResolvedTarget`) — the one entry point both LSP handlers (references/rename) and their CLI mirrors use to identify the target before calling `refs_to`. Never map `RenameKind`→`TargetRef` inline in a handler; per-feature policy on a target is a method on `TargetRef` (e.g. `supports_cross_file_rename`). Graph walking — one lazy `walk` over the visibility edges (inheritance / bridges / descendants) — landed: `docs/adr/graph-walking.md` (`GraphView`, the closed `EdgeKind` + exhaustive `edges_from`, the Model-layer placement, the file-role/lexical boundaries). Forward work in `docs/prompt-graph-walking.md`: the deferred Scope-node taxonomy, and **instance brands** (per-object dispatch scoping, multi-app Mojo) — spiked and PARKED as a downstream consumer of the long-distance value-provenance tier (`prompt-type-inference-residual.md`), not a standalone feature.

### Rules (read before writing code)

1. **All tree-sitter CST traversal happens inside `build()`.** No other file walks tree-sitter nodes, calls `child_by_field_name`, or uses `TreeCursor`. To add CST-derived data: extend `visit_*` in `builder.rs`. Builder plugins (separate modules taking `&mut FileAnalysis` + `&Tree` + `&[u8]`) are fine — they preserve the single entry point. Multiple post-walk passes inside `build()` are allowed and named (see "Build pipeline phases") for cases needing resolved state.

   **Within sanctioned tree consumers, speak `cst.rs`, not raw nodes.** `src/cst.rs` is the typed view over the CST (rust-analyzer style: zero-copy wrappers via `typed_node!` + `cast`, `NodeExt` for field-text/span/named-children, `pair_nodes` for separator-agnostic pair walking, `call_args` for flat call arguments, `varname_child`/`canonical_container_name` for variable identity, `is_conventional_invocant_scalar` for receiver detection). Each grammar trap is encoded there exactly once. New visitor code uses the typed accessors; re-spelling `child_by_field_name(..).utf8_text(..)` chains or a fresh `kind() == "..."` probe for a shape `cst.rs` already models is a bug. Name-level Perl conventions (`is_constructor_name`, `is_conventional_invocant_name`) live in `src/conventions.rs` — pure `&str`, importable by tree-free layers like `file_analysis.rs`.

2. **`file_analysis.rs` is the single source of truth.** All analysis results live in `FileAnalysis`. Query methods belong here. No `tree_sitter` imports.

3. **`symbols.rs` is a thin adapter** — `FileAnalysis` types → LSP types. No analysis, no tree walks, no Perl semantics decisions.

4. **`module_resolver.rs` calls the builder, then queries `FileAnalysis`** — never walks the tree directly.

5. **DRY: shared extraction logic goes on `FileAnalysis`.** Two callers needing the same data → one method, both call it.

6. **`cursor_context.rs` is the position-dependent exception** — gets a tree + source for completion/sig-help context. Does NOT modify `FileAnalysis`.

7. **Every meaningful token gets a ref.** If `ref_at(pos)` returns nothing or returns too-broad, the builder is missing emission. Overlapping refs → `ref_at` returns the **narrowest span**. Common gaps: fat-comma keys in calls (`connect(timeout => 30)` needs its own `HashKeyAccess`), hash literal keys, framework-synthesized entities (Moo `has name` → `HashKeyDef` for constructor, not just accessor).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tree-sitter-perl/perl-lsp](https://github.com/tree-sitter-perl/perl-lsp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
