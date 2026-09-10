---
trigger: always_on
description: Agentic working instructions for the **frunk** repository.  Read this before making changes.  It captures the architecture, the type-level techniques that make the library work, the invariants that must not be broken, and the exact commands CI enforces.
---

# AGENT.md

Agentic working instructions for the **frunk** repository.  Read this before making changes.  It captures the architecture, the type-level techniques that make the library work, the invariants that must not be broken, and the exact commands CI enforces.

## What frunk is

Frunk is a functional programming toolbelt for Rust: `HList`, `Coproduct`, `Generic`, `LabelledGeneric`, `Path` (lenses), `Validated`, `Semigroup` and `Monoid`.  The library's distinguishing feature is that most of it is implemented with **type-level programming** on stable Rust: no `unsafe`, no specialisation, no nightly-only features in the shipped code.  Correctness, type/memory safety and efficiency are the stated priorities, in that spirit.

Reference reading (the author's own write-ups, useful for understanding intent, not just mechanics):

* HLists: <https://beachape.com/blog/2016/10/23/rust-hlists-heterogenously-typed-list/>
* Generic: <https://beachape.com/blog/2017/02/04/rust-generic-not-generics/>
* LabelledGeneric: <https://beachape.com/blog/2017/03/04/labelledgeneric-in-rust-what-why-how/>
* Type-level recursion and sculpting: <https://beachape.com/blog/2017/03/12/gentle-intro-to-type-level-recursion-in-Rust-from-zero-to-frunk-hlist-sculpting/>
* Boilerplate-free struct transforms (transmogrify): <https://beachape.com/blog/2017/04/12/boilerplate-free-struct-transforms-in-rust/>
* Structural typing (paths): <https://beachape.com/blog/2021/05/25/structural-typing-in-rust/>

## Workspace layout

Frunk is a Cargo workspace.  Know which crate a change belongs in before editing.

* **`frunk`** (root, `src/`): the user-facing facade.  Re-exports `frunk_core` and `frunk_derives`, and adds `monoid`, `semigroup`, `validated`.  `#![no_std]` with an `alloc` feature.
* **`frunk_core`** (`core/`): the fundamental building blocks, intentionally minimal.  Modules: `hlist`, `coproduct`, `generic`, `labelled`, `path`, `indices`, `traits`, `tuples`, `macros`.  `#![no_std]` (+ `alloc`).
* **`frunk_derives`** (`derives/`): the `#[derive(Generic)]` and `#[derive(LabelledGeneric)]` custom derives.  `proc-macro = true`.
* **`frunk_proc_macros`** (`proc-macros/`): the `path!` and `Path!` function-like proc macros.
* **`frunk_proc_macro_helpers`** (`proc-macro-helpers/`): shared internals for the two proc-macro crates (type-level label encoding, HList/Coproduct AST builders, field-binding helpers).  Not published for direct use.
* **`frunk_laws`** (`laws/`): `quickcheck`-based property tests for the algebraic laws (`Semigroup`, `Monoid`).

Version note: `frunk`, `frunk_core`, `frunk_derives` share a version; `frunk_proc_macros`/`frunk_proc_macro_helpers` and `frunk_laws` are versioned independently.  Path dependencies also carry a `version` field, so a version bump in one crate must be propagated to its dependents.

## Build, test, lint

These are the exact checks CI (`.github/workflows/ci.yml`) runs on stable and nightly.  Run them locally before proposing a change; a green local run of these is the bar.

```sh
# formatting (must be clean)
cargo fmt --all -- --check

# clippy, warnings are errors, including test code
cargo clippy --all --tests -- -D warnings

# type-check and build (std)
cargo check
cargo build
cargo build --examples

# full test suite across the workspace (includes doctests)
cargo test --all --no-fail-fast

# docs must build clean, private items included
RUSTDOCFLAGS='-D warnings' cargo doc --all --no-deps --document-private-items
```

`no_std` is a first-class target and is checked in CI against an embedded target with default features off:

```sh
rustup target add thumbv6m-none-eabi
cargo check --target thumbv6m-none-eabi --no-default-features
cargo build --target thumbv6m-none-eabi --no-default-features
```

Benchmarks live in `benches/` and run with `cargo bench` (nightly-flavoured harness in CI).

## Core mental model

Internalise these five ideas; almost every file is an application of them.

### 1. HList = `HCons` / `HNil`

An `HList` is a heterogeneous, compile-time-typed list, structurally just an arbitrarily nested pair: `HCons<H, T>` (head + tail) terminated by `HNil`.  `Hlist![A, B, C]` is sugar for `HCons<A, HCons<B, HCons<C, HNil>>>`.  Construct with `hlist![a, b, c]`, pattern-match with `hlist_pat![a, b, c]`, and write the type with the `HList![..]` type macro.  All the interesting operations (`map`, `foldl`/`foldr`, `zip`, `sculpt`, `pluck`, `get`, `into_reverse`, `to_ref`/`to_mut`) are traits with a base case for `HNil` and a recursive case for `HCons`.

### 2. Type-level recursion driven by trait resolution, disambiguated by phantom index types


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lloydmeta/frunk](https://github.com/lloydmeta/frunk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
