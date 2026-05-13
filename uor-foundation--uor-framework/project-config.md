---
trigger: always_on
description: Rust workspace encoding the UOR Foundation ontology as typed data structures, a generated `#![no_std]` trait crate (`uor-foundation`), and validated serializations (JSON-LD, Turtle, N-Triples, OWL RDF/XML, JSON Schema, SHACL Shapes, EBNF). All source code, documentation, and web artifacts are machine-generated from the authoritative ontology defined in `spec/`.
---

# CLAUDE.md — UOR-Framework

## Project overview

Rust workspace encoding the UOR Foundation ontology as typed data structures, a generated `#![no_std]` trait crate (`uor-foundation`), and validated serializations (JSON-LD, Turtle, N-Triples, OWL RDF/XML, JSON Schema, SHACL Shapes, EBNF). All source code, documentation, and web artifacts are machine-generated from the authoritative ontology defined in `spec/`.

## Workspace layout

| Crate | Path | Published | Purpose |
|---|---|---|---|
| `uor-ontology` | `spec/` | no | Ontology source of truth (classes, properties, individuals, serializers) |
| `uor-codegen` | `codegen/` | no | Ontology-to-Rust trait generator |
| `uor-foundation` | `foundation/` | **crates.io** | Generated `#![no_std]` trait library — never edit manually |
| `uor-foundation-sdk` | `uor-foundation-sdk/` | **crates.io** (pending first release) | Procedural-macro ergonomics (`product_shape!`, `coproduct_shape!`, `cartesian_product_shape!`) for composing `ConstrainedTypeShape` operands — emitted by `uor-crate` from `codegen/src/sdk_macros.rs`. |
| `uor-conformance` | `conformance/` | no | Conformance suite (OWL, SHACL, RDF, Rust API, docs, website) — check count in `spec/src/counts.rs` |
| `uor-docs` | `docs/` | no | Documentation generator |
| `uor-website` | `website/` | no | Static site generator |
| `uor-lean-codegen` | `lean-codegen/` | no | Ontology-to-Lean 4 structure generator |
| `uor-clients` | `clients/` | no | CLI binaries: `uor-build`, `uor-crate`, `uor-lean`, `uor-docs`, `uor-website`, `uor-conformance` |
| `cargo-uor` | `cargo-uor/` | no | Cargo subcommand binary for UOR tooling |

## Critical rules

- **Never hand-edit `foundation/src/` or `lean4/`** — they are regenerated from `spec/` by `uor-crate` and `uor-lean`. CI enforces `git diff --exit-code` on both.
  - **Exception (Phase 11):** `foundation/src/blanket_impls.rs` is hand-written and starts with `// @codegen-exempt`. The codegen `emit::write_file` preserves files carrying that banner; the `rust/blanket_impls_exempt` conformance gate enforces both the banner and the required Path-3 blanket impls.
- **On release**, Lean 4 cloud release builds are uploaded via `lake upload`. Lean Reservoir indexes this repo directly (root `lakefile.lean` + `lake-manifest.json`).
- **All clippy warnings are errors.** CI runs `cargo clippy --all-targets -- -D warnings`.
- **Every crate denies:** `clippy::unwrap_used`, `clippy::expect_used`, `clippy::panic`, `missing_docs`, `clippy::missing_errors_doc`.
- **Formatting is enforced.** CI runs `cargo fmt --check`.
- **The conformance suite must pass.** `cargo run --bin uor-conformance` — zero failures allowed (check count in `spec/src/counts.rs`).
- **No `unsafe` code.** The `uor-foundation` crate is `#![no_std]` with zero dependencies.
- **Bracket-escape doc comments.** Use `normalize_comment()` to prevent rustdoc intra-doc link warnings on `[text]` in comments.

## Build commands

```sh
cargo fmt --check                    # Format check
cargo clippy --all-targets -- -D warnings  # Lint
cargo test                           # Unit + integration tests
cargo run --bin uor-crate            # Regenerate foundation/src/ from spec/
cargo run --bin uor-lean             # Regenerate lean4/ from spec/
cargo run --bin uor-build            # Emit JSON-LD, Turtle, N-Triples to public/
cargo run --bin uor-docs             # Generate documentation site
cargo run --bin uor-website          # Generate website
cargo run --bin uor-conformance      # Run full conformance suite
```

Docs/website/conformance binaries accept `PUBLIC_BASE_PATH` env var for URL prefixing.

## CI pipeline (in order)

`cargo fmt --check` → `cargo clippy` → `cargo test` → `cargo run --bin uor-crate` → `git diff --exit-code foundation/src/ uor-foundation-sdk/src/` → `cargo check -p uor-foundation --no-default-features` → `cargo publish --dry-run` (uor-foundation + uor-foundation-sdk) → `uor-lean` → `git diff --exit-code lean4/` → `uor-build` → `uor-docs` → `uor-website` → `uor-conformance` → deploy pages

## Ontology architecture

Counts below are mirrored from `spec/src/counts.rs`, which is the single source of truth.

- **34 namespaces**, assembly order: `u → schema → op → query → resolver → type → partition → foundation → observable → carry → homology → cohomology → proof → derivation → trace → cert → morphism → state → reduction → convergence → division → interaction → monoidal → operad → effect → predicate → parallel → stream → failure → linear → recursion → region → boundary → conformance`
- **Space classification:** Kernel (17: `u`, `schema`, `op`, `carry`, `reduction`, `convergence`, `division`, `monoidal`, `operad`, `effect`, `predicate`, `parallel`, `stream`, `failure`, `linear`, `recursion`, `region`), Bridge (14: `query`, `resolver`, `partition`, `foundation`, `observable`, `homology`, `cohomology`, `proof`, `derivation`, `trace`, `cert`, `interaction`, `boundary`, `conformance`), User (`type`, `morphism`, `state`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UOR-Foundation/UOR-Framework](https://github.com/UOR-Foundation/UOR-Framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
