---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Thinking & Quality

Always use maximum thinking effort. Never rush to completion. Read code thoroughly before modifying. When investigating bugs, verify assumptions by reading the actual code rather than guessing. Test locally before publishing or deploying. Never use `git add -A` — always stage specific files by name.

## Project Overview

**wasmsh** is a shell runtime written in Rust that targets two in-process WebAssembly platforms plus a scalable server-side deployment path from a shared core:

1. **Standalone** (`wasm32-unknown-unknown`) — browser Web Worker via `wasm-bindgen`
2. **Pyodide** (`wasm32-unknown-emscripten`) — linked into a custom Pyodide build, sharing the Python interpreter's Emscripten module and filesystem
3. **Scalable** (Kubernetes) — `wasmsh-dispatcher` (Rust HTTP control plane) plus a pool of `wasmsh-runner` pods (Node + Pyodide) installed via `deploy/helm/wasmsh`. Clients speak JSON/HTTP to the dispatcher; the `langchain-wasmsh` adapters ship `WasmshRemoteSandbox` as the first-party client. See `docs/explanation/snapshot-runner.md`.

Execution pipeline: `source -> lexer -> parser -> AST -> HIR -> runtime executor`.

The runtime currently uses two execution paths:
- Default path: direct HIR interpretation in `wasmsh-runtime`
- VM subset path: selected top-level `and/or` lists lower through `wasmsh-ir` into `wasmsh-vm`

Expansion, redirection planning, dispatch, budgeting, and protocol emission are owned by the runtime layer and shared by both paths.

Code, comments, and documentation are in English.

## Current State

The repository has multi-layer coverage across crate tests, runtime/protocol tests, TOML suite cases, and E2E adapters. The runtime/protocol crates are expected to stay green; the broad TOML suite still contains known conformance gaps in areas like arrays, brace expansion, globbing, and `xtrace`. See `SUPPORTED.md` for syntax/command coverage.

**Standalone path**: `wasmsh-browser` wraps `wasmsh-runtime` with wasm-bindgen glue. 15 Playwright E2E tests.

**Pyodide path**: `wasmsh-pyodide` wraps `wasmsh-runtime` with C ABI + JSON protocol. `EmscriptenFs` backend routes VFS through libc (shared with Python). `python`/`python3` commands run in-process via `PyRun_SimpleString`. `pip install` is intercepted at the JS host and routed through micropip. Both Node and browser use `loadPyodide()` for boot. 76 Node E2E tests (15 suites) + 21 Playwright browser E2E tests.

**Scalable path**: `crates/wasmsh-dispatcher` is an Axum HTTP control plane with session affinity, restore-capacity routing, drain, and Prometheus metrics. Runner pods run `tools/runner-node/src/server.mjs` (Node + the Pyodide path above) and expose `/readyz`, `/runner/snapshot`, `/sessions/...`. Deployment lives in `deploy/helm/wasmsh` (HPA, PDB, NetworkPolicy, headless service) with production images `ghcr.io/mayflower/wasmsh-{dispatcher,runner}`. End-to-end coverage: `e2e/dispatcher-compose` (docker-compose, fast) and `e2e/kind` (full Helm install in kind).

Notable features: `[[ ]]`, `(( ))`, C-style `for (( ))`, arrays, `declare`/`typeset`, `alias`/`unalias`, `let`, `shopt`, extended globbing, globstar, full arithmetic, case modification, indirect expansion, dynamic variables (`$RANDOM`, `$LINENO`, `$SECONDS`, `$FUNCNAME`, `$BASH_SOURCE`, `$PIPESTATUS`), `printf`/`read`, `mapfile`, `builtin`, `select`, `|&`, `case` fall-through, `set -euo pipefail`, 88 utilities (jq, awk, yq, bc, rg, fd, diff/patch, tree, tar, gzip, unzip, xxd, dd, strings, md5sum/sha*sum, curl, wget).

## Rust Toolchain

Pinned via `rust-toolchain.toml` (stable + rustfmt, clippy, rust-src, llvm-tools, `wasm32-unknown-unknown` + `wasm32-unknown-emscripten` targets). Cargo may not be on PATH by default:
```bash
export PATH="$HOME/.rustup/toolchains/stable-aarch64-apple-darwin/bin:$PATH"
```

## Build & Test Commands

```bash
# ── Core ────────────────────────────────────────────
just check                  # fmt-check + clippy + fast test (pre-push)
just ci                     # full CI: fmt + clippy + test + deny + doc
just test                   # all Rust tests (nextest or cargo test)
just test-suite             # TOML declarative test suite only
just test-crate wasmsh-lex  # single crate

# ── Standalone ──────────────────────────────────────
just build-standalone       # wasm-pack → e2e/standalone/fixture/pkg/
just test-e2e-standalone    # Playwright browser E2E (15 tests)

# ── Pyodide (requires emcc) ────────────────────────
just build-pyodide          # custom Pyodide → dist/pyodide-custom/
just test-e2e-pyodide-node  # Node E2E (76 tests, 15 suites)
just test-e2e-pyodide-browser # Playwright browser E2E (21 tests)
just build-emscripten-probe # emscripten staticlib probe

# ── Scalable dispatcher + runner ────────────────────
just test-e2e-dispatcher-compose  # docker-compose e2e (fast local loop)
just test-e2e-kind                # kind + Helm e2e (production parity)

# ── Quality ─────────────────────────────────────────
just clippy-wasm            # clippy for wasm32 target
just coverage               # HTML coverage report

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mayflower/wasmsh](https://github.com/mayflower/wasmsh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
