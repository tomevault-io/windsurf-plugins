---
trigger: always_on
description: handles; they do not own the domain operation or interface composition.
---

# ALUX Rust repository guide

This file is the compact engineering authority for humans and coding agents working on ALUX Rust.
Apply it to the whole repository.

## Product

ALUX Rust provides reusable Design by Meaning infrastructure for independently published Rust
specifications. It reifies derived operations as first-order values and describes typed HTTP and
JSON-RPC programs independently of their concrete frameworks.

This repository is specification infrastructure. Do not turn it into a central domain-spec crate,
application framework, service runtime, dependency-injection container, or framework-owned API
generator without an explicit scope decision. Downstream crates own their domain capabilities.

## Workspace

| Crate | Responsibility |
| --- | --- |
| `alux-ext` | `ext` facade, first-order operation signature/application, and owned context handles |
| `alux-ext-macros` | Parsing, validation, and lowering for extension, HTTP, and JSON-RPC programs |
| `alux-http` | Neutral typed HTTP syntax, algebras, and folds |
| `alux-jsonrpc` | Neutral typed JSON-RPC syntax, algebras, and folds |
| `alux-http-text` | Text and metadata interpretation of an HTTP program |
| `alux-http-poem` | Poem interpretation of an HTTP program |
| `alux-jsonrpc-jsonrpsee` | jsonrpsee interpretation of a JSON-RPC program |

Dependency arrows point toward `alux-ext`. The specification crates declare exactly one dependency,
`alux-ext`, and re-export the macro backends they own from `alux_ext::macros`.

```text
alux-http-text ------\
                      -> alux-http ------\
alux-http-poem ------/                    -> alux-ext -> alux-ext-macros
alux-jsonrpc-jsonrpsee -> alux-jsonrpc --/
```

A specification crate must never name a framework, serialization format, or runtime. Each
interpretation is a separate package named `alux-<program>-<interpreter>`, and adding one must not
require changing the program it folds.

## Authority

Read authority in this order:

1. The semantic role documented for an algebra or first-order program defines the intended meaning.
2. Public trait signatures and syntax types define primitive executable meaning.
3. Extension bounds, generated operation signatures, and generic folds define derived meaning.
4. Public laws and shared scenarios define reusable obligations.
5. Text, Poem, jsonrpsee, and downstream interpreters witness the meaning.
6. Procedural-macro expansion implements a projection of authored code; it does not redefine it.
7. Prose summarizes the artifacts above.

If these disagree, do not silently choose the framework or macro expansion. State the mismatch and
fix the smallest authoritative layer that is wrong.

## Read order

For design work, read from meaning to machinery:

1. `README.md`
2. `DENOTATIONAL_DESIGN.md`
3. `ARCHITECTURE.md`
4. `alux-ext/src/lib.rs`
5. `alux-http/src/algebra.rs`, `output.rs`, and `program.rs` for HTTP work
6. `alux-jsonrpc/src/algebra.rs` and `program.rs` for JSON-RPC work
7. the relevant interpreter crate
8. `alux-ext-macros` only after the first-order target meaning is clear
9. tests that compare interpretations

Do not begin with generated tokens or a framework callback merely because it is concrete.

## Commands

The CI-equivalent commands are:

```sh
cargo fmt --all -- --check
cargo build --workspace --all-features --all-targets
cargo clippy --workspace --all-features --all-targets -- -D warnings
RUSTDOCFLAGS="-D warnings" cargo doc --workspace --all-features --no-deps
cargo nextest run --workspace --all-features --no-fail-fast
cargo test --workspace --all-features --doc
```

The `Justfile` provides these workflows. Run the narrowest relevant test during development and
`just ci` before finishing.

## Design by Meaning

This is a Rust adaptation of Conal Elliott's Denotational Design, taught in full in the
[ALUX programming guidelines](https://alux-network.github.io/alux-programming/). Read the lineage,
limits, and review rules in `DENOTATIONAL_DESIGN.md`; do not reduce the method to “use traits and
macros.”

- Ask what value, observation, transformation, or composition is specified before choosing syntax,
  storage, callbacks, framework types, or generated code.
- Define primitive domain meaning in small downstream capability traits.
- Put derived behavior in `#[ext]` implementations over explicit `where` clauses.
- Treat an extension's bounds as its semantic dependency declaration.
- Reify a method only when its application must become first-order data.
- Preserve context, argument product, argument names, and output as operation meaning.
- Describe transport surfaces as neutral program trees before selecting a framework.
- Interpret one program into execution, documentation, metadata, tests, or another backend.
- Keep concrete interpreters thin. They own extraction, conversion, registration, and runtime
  handles; they do not own the domain operation or interface composition.
- Keep macro lowering syntax-directed. It must not infer domain policy or invent hidden routes,
  methods, arguments, or outputs.
- Add a primitive only when new meaning is required. Add syntax only when new composition must be
  preserved for interpreters.

Target shape:

```text
downstream capability traits
    -> derived extension methods

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alux-network/alux-rust](https://github.com/alux-network/alux-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
