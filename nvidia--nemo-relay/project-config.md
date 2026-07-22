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

NeMo Relay is a multi-language agent runtime framework for execution scopes, lifecycle events, middleware, plugins, and observability around tool and LLM calls. The core runtime is Rust. Primary supported bindings are Rust, Python, and Node.js. Go and the raw C FFI are experimental and source-first.

The shared runtime model is:

1. Scope stacks decide where work belongs and which scope-local behavior is visible.
2. Middleware registries decide what guardrails and intercepts run around managed calls.
3. Plugins install reusable runtime behavior from configuration.
4. Events record runtime behavior in ATOF form.
5. Subscribers and exporters consume events in-process or export them to ATIF, OpenTelemetry, OpenInference, or other backends.

## Repository Structure

The repository layout separates the Rust runtime, language bindings,
documentation, integrations, and agent-facing skills.

```text
crates/
  core/       # Rust core runtime crate, published as nemo-relay
  adaptive/   # Adaptive runtime primitives and plugin components
  python/     # PyO3 native extension for the Python package
  ffi/        # Raw C ABI layer used by downstream bindings such as Go
  node/       # NAPI Node.js binding and JavaScript/TypeScript entry points
python/
  nemo_relay/  # Python wrapper package: scopes, tools, LLM, middleware, typed helpers, plugins, adaptive helpers
  tests/      # Python tests
go/
  nemo_relay/  # Experimental Go CGo binding and tests
fern/         # Fern documentation site
scripts/      # Stable wrappers and helper scripts; build/test/docs entry points live in justfile
skills/       # Published Codex/agent skills for NeMo Relay usage patterns
```

## Prerequisites

Install the tools needed for the surfaces you touch. For a full repository validation environment, install all of these:

| Tool | Version / Notes | Required For |
|---|---|---|
| Rust | Docs minimum is 1.86 or newer; the repo pins the active toolchain in `rust-toolchain.toml` | Rust core, native bindings, FFI |
| Python | 3.11 or newer | Python package, PyO3 builds, docs tooling |
| Node.js | 24 or newer, with npm | Node.js binding, generated API docs |
| Go | 1.21 or newer | Experimental Go binding |
| `uv` | Current project workflow tool | Python environments, docs dependencies, pre-commit |
| `just` | 1.40 or newer | Canonical build, test, docs, package task runner |
| `cargo-deny` | Current stable | Rust dependency auditing |
| `cargo-nextest` | 0.9.111 or newer | CI-style Rust test runs |
| `cargo-llvm-cov` | 0.8.5 or newer | CI-style coverage reports |

Common setup commands:

```bash
cargo install just --locked
cargo install cargo-deny --locked
cargo install cargo-nextest --version 0.9.111 --locked
cargo install cargo-llvm-cov --version 0.8.5 --locked

uv sync
uv run pre-commit install

npm install --ignore-scripts
```

`uv sync` installs Python development and test dependencies, including `maturin`, `ruff`, `ty`, and `pre-commit`. Documentation recipes sync the docs dependency group as needed, but Python, Node.js, npm, `uv`, and `just` still need to exist on PATH.

## Build, Test, And Docs Commands

Prefer the repository `just` recipes over raw tool commands. Use raw `cargo`, `pytest`, `go test`, or `npm` commands only for focused debugging or targeted single-test reruns that do not have a `just` recipe.

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
just build-all
```

Test targets:

```bash
just test-rust
just ci=true test-rust       # CI-style Rust test run; uses nextest and coverage tooling when available
just test-python
just test-node
just test-go
just test-all
```

Documentation targets:

```bash
just docs
just docs-api-reference
```

Package targets:

```bash
just package-python
just package-node
```

Cleanup:

```bash
just clean
```

Focused fallback commands are acceptable for narrow loops:

```bash
cargo test -p nemo-relay -- <test_name>
uv run pytest python/tests/test_scope.py
uv run pytest -k "test_name"
cd crates/node && node --test --test-name-pattern="pattern" tests/*.mjs
cd go/nemo_relay && go test -v -run TestFoo ./...
```

## Validation Expectations

Run tests for every language affected by a change. If you touch the Rust core runtime, middleware semantics, event shape, scope behavior, typed codecs, plugins, or observability, expect to validate every affected binding because the bindings share the same runtime contract.

Minimum guidance:

- Rust core or adaptive changes: `just test-rust`; add binding tests when public behavior changes.
- Python binding or wrapper changes: `just test-python`.
- Node.js binding or wrapper changes: `just test-node`.
- Go binding or raw FFI changes: `just test-go` and the relevant Rust/FFI checks.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/NeMo-Relay](https://github.com/NVIDIA/NeMo-Relay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
