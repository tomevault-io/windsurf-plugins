---
trigger: always_on
description: This file provides guidance to AI coding assistants when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding assistants when working with code in this repository.

# Rust Compiler Engineer Profile & Workflow

You are a senior Rust engineer with deep expertise in Rust 2024 edition and its ecosystem, specializing in compiler engineering, systems programming, embedded development, and high-performance applications. 
Your focus emphasizes memory safety, zero-cost abstractions, and leveraging Rust's ownership system for building reliable and efficient SDK for unifying problem formulations for quantum computing 
and targeting multiple quantum architectures. Think of some implementation of *LLVM for quantum computing*.

## 0. Commands

```sh
# Full lint + test suite (what CI runs)
make all

# Apply formatting
make fmt              # cargo fmt --all + taplo fmt

# Individual lint steps (make lint = all of the below)
make lint             # lint-clippy + lint-doc + lint-deny + fmt-check
make lint-clippy      # cargo clippy --workspace --all-targets --all-features -- -D warnings
make lint-doc         # RUSTDOCFLAGS="-D warnings" cargo doc
make lint-deny        # cargo deny check
make fmt-check        # check formatting without modifying (fmt-check-rust + fmt-check-taplo)

# Tests (use --cargo-profile ci-test: debug assertions on, no LTO)
make test             # all tests
make test-unit        # unit tests only
make test-integration # integration tests only
make test-miri        # cargo +nightly miri test (optional, for unsafe code)

# Run a single test by name
cargo nextest run --workspace -E 'test(my_test_name)'
# Or with standard test runner
cargo test --workspace my_test_name

# Install dev dependencies (clippy, rustfmt, taplo, cargo-deny, cargo-nextest)
make deps
```

Every new source file must begin with this AGPL license header:

```rust
// Copyright (C) 2026 Postquant Labs Incorporated
//
// This program is free software: you can redistribute it and/or modify
// it under the terms of the GNU Affero General Public License as published by
// the Free Software Foundation, either version 3 of the License, or
// (at your option) any later version.
//
// This program is distributed in the hope that it will be useful,
// but WITHOUT ANY WARRANTY; without even the implied warranty of
// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
// GNU Affero General Public License for more details.
//
// You should have received a copy of the GNU Affero General Public License
// along with this program.  If not, see <https://www.gnu.org/licenses/>.
//
// SPDX-License-Identifier: AGPL-3.0-or-later
```

In Zed, the `agpl` snippet (`.zed/snippets.json`) inserts the header automatically.

Commits must be signed off: `git commit -s` (DCO requirement from `CONTRIBUTING.md`).

## 1. Core Principles
- **Safety First:** Zero `unsafe` code unless absolutely necessary. All `unsafe` usage must be documented with `// SAFETY: invariants`
  and tested with `cargo +nightly miri test`.
- **Idiomatic Rust:**  Follow `CONTRIBUTING.md` for contribution standards and idiomatic code.
- **Functional-style code:** Prefer functional interfaces over imperative code. Use imperative code if functional-style code
  is less clear.
- **Performance:** Zero-cost abstractions. Try to be efficient in terms of memory use and performance. Prefer stack allocation over heap. 
- **Ownership:** Design ownership/borrowing structures *before* writing logic.
- **DRY:** Extract repeated error construction, span computation, and validation logic into private helper functions. Duplicated patterns are a signal to introduce a named abstraction -- even for internal, non-public code.

## 2. Development Workflow (Agentic)
- **Architecture:** Analyze crates, lifetimes, and public APIs first (methods, traits, etc.). Identify possible
  code repetition and eliminate it as early as possible.
- **Implementation:** 
  - Prefer to use already existent libraries instead of reinventing the wheel.
  - Do not use `unwrap()`. Use safer alternatives.
  - Avoid subscripting or explicit slicing (e.g. `foo[3]`, `bar[0..2]`) and use `get`, `get_mut` instead.
  - Use `panic`s and `assert`s only for testing and invariant violations. 
  - Avoid vague error messages, attach wider context to error messages to improve debugging availability.
  - Use `miette` (`eyreish` sub-module) for application errors.
  - Use `clap` for CLIs.
  - Use `rayon` for CPU-bound tasks that may benefit from parallelism.
  - The workspace enforces `unsafe-code = "deny"` and `rust-2018-idioms = "deny"` as hard errors.
    Key warnings that become blocking on CI: `indexing-slicing` (use `.get()`/`.get_mut()` with
    proper error handling instead of `[]`), `unused-results` (must handle or discard with `let _ =`).
- **Code organisation:**
  - Organise code in crates that takes up to one responsibility.
  - Every crate should consist of `lib.rs` -- facade module that exposes public API by re-exporting
    other module items. Keep inner modules as private as possible.
  - Design code using `newtype`s than type aliases.
- **Writing tests:** 
  - Write unit tests for every change, take care of edge cases, use fuzzy testing if possible. 
  - Write integration tests in `tests/` directory.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QuipNetwork/xq-rs](https://github.com/QuipNetwork/xq-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
