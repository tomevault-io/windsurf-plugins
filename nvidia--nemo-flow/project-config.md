---
trigger: always_on
description: SPDX-FileCopyrightText: Copyright (c) 2026, NVIDIA CORPORATION & AFFILIATES. All rights reserved.
---

<!--
SPDX-FileCopyrightText: Copyright (c) 2026, NVIDIA CORPORATION & AFFILIATES. All rights reserved.
SPDX-License-Identifier: Apache-2.0
-->

# AGENTS.md

This file provides guidance to agents, including Claude Code and OpenAI Codex, when working in this repository.

## Project Overview

NeMo Flow is a multi-language agent runtime framework for execution scopes, lifecycle events, middleware, plugins, and observability around tool and LLM calls. The core runtime is Rust. Primary supported bindings are Rust, Python, and Node.js. Go, WebAssembly, and the raw C FFI are experimental and source-first.

The shared runtime model is:

1. Scope stacks decide where work belongs and which scope-local behavior is visible.
2. Middleware registries decide what guardrails and intercepts run around managed calls.
3. Plugins install reusable runtime behavior from configuration.
4. Events record runtime behavior in ATOF form.
5. Subscribers and exporters consume events in-process or export them to ATIF, OpenTelemetry, OpenInference, or other backends.

## Repository Structure

The repository layout separates the Rust runtime, language bindings, documentation,
integration patches, and agent-facing skills.

```text
crates/
  core/       # Rust core runtime crate, published as nemo-flow
  adaptive/   # Adaptive runtime primitives and plugin components
  python/     # PyO3 native extension for the Python package
  ffi/        # Raw C ABI layer used by downstream bindings such as Go
  node/       # NAPI Node.js binding and JavaScript/TypeScript entry points
  wasm/       # wasm-bindgen WebAssembly binding and JS wrappers
python/
  nemo_flow/  # Python wrapper package: scopes, tools, LLM, middleware, typed helpers, plugins, adaptive helpers
  tests/      # Python tests
go/
  nemo_flow/  # Experimental Go CGo binding and tests
docs/         # Sphinx documentation site
scripts/      # Stable wrappers and helper scripts; build/test/docs entry points live in justfile
third_party/  # Pinned upstream checkouts for sample integration patches
patches/      # NeMo Flow patch sets applied to third_party checkouts
skills/       # Published Codex/agent skills for NeMo Flow usage patterns
```

## Prerequisites

Install the tools needed for the surfaces you touch. For a full repository validation environment, install all of these:

| Tool | Version / Notes | Required For |
|---|---|---|
| Rust | Docs minimum is 1.86 or newer; the repo pins the active toolchain in `rust-toolchain.toml` | Rust core, native bindings, FFI, WebAssembly |
| Python | 3.11 or newer | Python package, PyO3 builds, docs tooling |
| Node.js | 20 or newer, with npm | Node.js binding, WebAssembly JS tests, generated API docs |
| Go | 1.21 or newer | Experimental Go binding |
| `uv` | Current project workflow tool | Python environments, docs dependencies, pre-commit |
| `just` | 1.40 or newer | Canonical build, test, docs, package task runner |
| `wasm-pack` | 0.14.0 or newer | WebAssembly build and integration tests |
| `cargo-deny` | Current stable | Rust dependency auditing |
| `cargo-nextest` | 0.9.111 or newer | CI-style Rust test runs |
| `cargo-llvm-cov` | 0.8.5 or newer | CI-style coverage reports |

Common setup commands:

```bash
cargo install just --locked
cargo install cargo-deny --locked
cargo install cargo-nextest --version 0.9.111 --locked
cargo install cargo-llvm-cov --version 0.8.5 --locked
cargo install wasm-pack --version 0.14.0 --locked

uv sync
uv run pre-commit install

cd crates/node
npm install --ignore-scripts
```

`uv sync` installs Python development and test dependencies, including `maturin`, `ruff`, `ty`, and `pre-commit`. Documentation recipes sync the docs dependency group as needed, but Python, Node.js, npm, `uv`, and `just` still need to exist on PATH.

## Build, Test, And Docs Commands

Prefer the repository `just` recipes over raw tool commands. Use raw `cargo`, `pytest`, `go test`, `npm`, or `wasm-pack` commands only for focused debugging or targeted single-test reruns that do not have a `just` recipe.

Discover the current task surface with:

```bash
just --list
```

Build targets:

```bash
just build-rust
just build-python
just build-node
just build-go
just build-wasm
just build-all
```

Test targets:

```bash
just test-rust
just ci=true test-rust       # CI-style Rust test run; uses nextest and coverage tooling when available
just test-python
just test-node
just test-go
just test-wasm
just test-all
```

Documentation targets:

```bash
just docs
just docs-linkcheck
just docs-github-pages
```

Package targets:

```bash
just package-python
just package-node
just package-wasm
```

Cleanup:

```bash
just clean
```

Focused fallback commands are acceptable for narrow loops:

```bash
cargo test -p nemo-flow -- <test_name>
uv run pytest python/tests/test_scope.py
uv run pytest -k "test_name"
cd crates/node && node --test --test-name-pattern="pattern" tests/*.mjs
cd go/nemo_flow && go test -v -run TestFoo ./...
wasm-pack test --node crates/wasm
```

## Validation Expectations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/NeMo-Flow](https://github.com/NVIDIA/NeMo-Flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
