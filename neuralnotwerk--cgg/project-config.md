---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`cgg` is a CLI that generates call graphs (mermaid by default; also json/dot/graphml) from source trees. It is offline, deterministic, single-binary — no language servers, no build artifacts required. Supports 44 languages via tree-sitter plugins (including the Smithy, Protobuf, GraphQL, OpenAPI/Swagger, and AsyncAPI interface/descriptor languages, whose shape graphs are mapped onto the call-graph model), plus Jupyter notebooks (`.ipynb`) via a JSON cell extractor that feeds the Python plugin. OpenAPI/AsyncAPI documents are YAML or JSON, both parsed with the YAML grammar and content-detected by their root `openapi:`/`swagger:`/`asyncapi:` key (see `cgg-lang::detect`), so ordinary `.yaml`/`.json` files are untouched. Primary consumer of the output is coding agents reading mermaid in their context window.

## Commands

```bash
# Build the CLI (workspace member `cgg`)
cargo build --release -p cgg

# Run the full test suite — this is what the pre-commit hook gates on
cargo test --workspace

# Run a single crate's tests
cargo test -p cgg-resolve

# Run a single test by name
cargo test -p cgg-lang detect_python_by_shebang

# Run cgg against itself (sanity check)
./target/release/cgg ./crates -t mermaid --filter 'cgg::run$' -n 1

# Reproduce the README benchmark table (clones repos into $CGG_BENCH_DIR, default /storage/cgg-test_repos)
./scripts/benchmark.sh

# Install the project pre-commit hook (test + release build + README regen)
./scripts/install-hooks.sh

# Bypass the pre-commit hook for a docs-only commit
CGG_SKIP_PRECOMMIT=1 git commit ...    # or: git commit --no-verify
```

There is no separate lint step configured beyond `cargo`'s built-in checks; `deny.toml` exists for `cargo-deny` license/advisory auditing.

## Architecture

The workspace is a strict pipeline. Data flows one direction; later crates depend on earlier ones, never the reverse.

```text
cgg-walk  →  cgg-lang  →  cgg-resolve  →  cgg (query)  →  cgg-format
                                              ↑
                                          cgg-core (Graph, IDs, audit — shared types)
```

- **cgg-core** — the substrate. `Graph`, callable/edge IDs, audit records, facts, stdlib lookup tables. Every other crate depends on this; it depends on nothing internal.
- **cgg-walk** — file discovery. Honors `.gitignore` + a built-in deny list, classifies files (binary detection, symlink-chain guards), emits `WalkOutcome`.
- **cgg-lang** — language plugin layer. `LanguageDetector` (extension/shebang/header), `ParserPool` (tree-sitter parser caching), and `PluginRegistry::with_v1_plugins` which wires in all 44 `LanguagePlugin` impls under `crates/cgg-lang/src/plugins/`. `.ipynb` files are pre-processed in `cgg-lang::notebook::extract_python_source` before being handed to the Python plugin. Each plugin implements `extract` to pull callables + raw call sites out of a tree-sitter AST.
- **cgg-resolve** — links call sites to definitions. The order in `cgg::run` matters:
  1. `type_hints::build_return_type_map` + `propagate_types_with_returns` — infer variable types from params, locals (file-wide, conflict-aware), constructors, return types
  2. `intra_file::link_file` — scope/containment within a single file, with owner-qualified disambiguation of same-name candidates (`names::owner_from_qn`, handling `Self`/qualifier owners)
  3. `stack_graphs_resolver::resolve` (or `resolve_light` fallback on timeout, controlled by `--stack-graphs auto|on|off`) — precise name resolution where available
  4. `cross_file::resolve` — import chains, `#include` transitive closure (depth 8), pub-use chains, and an `(language, owner type, method)` index for typed-receiver method calls (replaces the old O(callables) suffix-scan)
  5. `ffi::link_ffi` — PyO3 / wasm-bindgen / napi / JNI / P/Invoke / `extern "C"` cross-language edges
  6. `dispatch::fanout` (opt-in, `--dynamic-dispatch`) — interface/trait declaration → implementation edges (`Via::Dynamic`), driven by `CallableNode::trait_impl_target`
  `names.rs` holds the shared `owner_from_qn` used by intra_file and cross_file. Every edge carries a confidence level and resolver provenance, so downstream consumers can filter by quality.
- **cgg** (the binary) — `main.rs` → `run` orchestrates the pipeline; `cli.rs` parses flags; `query.rs` applies `--filter` + `-n` (BFS neighborhood / path extraction) and `--exclude-*`. `since.rs` resolves `--since <revspec>` by shelling out to `git diff` and intersecting changed line ranges with callable spans; the resulting qualified names are appended to `--filter` as `^name$` regexes before `apply_query` runs. `synthesize_exit_nodes` mints the deduplicated external/stdlib leaf nodes for `--include-external`/`--include-stdlib` (after the audit-reconciliation prune, before `dedup_edges`). All four of `--include-external`/`--include-stdlib`/`--dynamic-dispatch`/`--reference-edges` are opt-in and never change the default graph.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NeuralNotwerk/cgg](https://github.com/NeuralNotwerk/cgg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
