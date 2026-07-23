---
trigger: always_on
description: Noir is a domain-specific language for SNARK proving systems (zero-knowledge proofs). The compiler is written in Rust and produces ACIR (Abstract Circuit Intermediate Representation), which can be consumed by any ACIR-compatible proving backend. The project also includes a CLI (`nargo`), LSP server, debugger, formatter, and JavaScript/WASM bindings.
---

# Noir Monorepo Development Guide

## Project Overview

Noir is a domain-specific language for SNARK proving systems (zero-knowledge proofs). The compiler is written in Rust and produces ACIR (Abstract Circuit Intermediate Representation), which can be consumed by any ACIR-compatible proving backend. The project also includes a CLI (`nargo`), LSP server, debugger, formatter, and JavaScript/WASM bindings.

## Architecture

### Compilation Pipeline

```
Source Code → [Lexing] → Tokens → [Parsing] → AST → [Name Resolution + Type Checking (Elaboration)] → HIR → [Monomorphization] → Monomorphized AST → [SSA Generation] → SSA → [SSA Optimizations] → ACIR/Brillig
```

### Workspace Structure

**Compiler** (`compiler/`):
- `noirc_frontend` — Lexer, parser, elaborator (name resolution + type checking), monomorphization. Entry point for the frontend pipeline.
- `noirc_evaluator` — SSA generation, SSA optimization passes, ACIR generation, Brillig generation. The middle/back-end.
- `noirc_driver` — Orchestrates the full compilation pipeline from source to artifacts.
- `fm` — File manager abstraction for source file handling.
- `noirc_errors` — Error reporting with source spans.

**ACVM** (`acvm-repo/`):
- `acir` — Circuit intermediate representation (analogous to LLVM IR for circuits).
- `brillig` — Bytecode format for unconstrained (non-deterministic) execution.
- `acvm` — Virtual machine that executes ACIR circuits.
- `brillig_vm` — Virtual machine that executes Brillig bytecode.
- `blackbox_solver`, `bn254_blackbox_solver` — Cryptographic primitives (hash functions, elliptic curve ops).

**Tooling** (`tooling/`):
- `nargo_cli` — Main CLI tool. Also hosts integration test harness (`tests/execute.rs`, `tests/stdlib-tests.rs`).
- `nargo` — Package manager core (dependency resolution, workspace handling).
- `lsp` — Language Server Protocol implementation.
- `nargo_fmt` — Code formatter.
- `noirc_abi` — ABI handling (conversion between JSON/TOML inputs and Noir types).

**Standard Library** (`noir_stdlib/`): Pure Noir implementations of stdlib functions (arrays, hashing, crypto, etc.).

### Test Programs

`test_programs/` contains integration test suites organized by expected outcome:
- `execution_success/` — Programs that should execute successfully (have `Prover.toml` inputs).
- `execution_failure/` — Programs that should fail at runtime.
- `compile_failure/` — Programs that should fail to compile.
- `compile_success_empty/` — Programs that compile to empty circuits.
- `compile_success_contract/` — Smart contract compilation tests.

Test cases are auto-generated from these directories by `tooling/nargo_cli/build.rs`.

### Key Patterns

- **Unsafe code is forbidden** (`#![forbid(unsafe_code)]` workspace-wide).
- **SSA passes** live in `compiler/noirc_evaluator/src/ssa/opt/` — each module has its own unit tests.
- **Elaboration** (`compiler/noirc_frontend/src/elaborator/`) combines name resolution and type checking in a single pass.
- PRs are **squash-merged** into `master`.

### Code Comments

A code comment must stand on its own when read against the post-change state of the file. Do not reference transitions between states ("was a `debug_assert!`, now a plain `assert!`"), the current task or fix ("added for the Y flow"), or what the code used to do. That context belongs in the commit message and PR description, where it naturally lives alongside the diff. In the code it rots the moment the PR stops being recent, leaving a reader with a narrative they cannot verify.

Test: imagine cherry-picking this file into a fresh repository with no git history. If the comment would still make sense to that reader, keep it. If it only makes sense to someone who remembers the change that introduced it, move it to the commit message.

## Design Decisions

The `design/` directory records specific design decisions for the Noir language, compiler, and
tooling (e.g. `design/comptime.md`, `design/oracles.md`). It is a record of why things work the
way they do, not user-facing documentation (that lives in `docs/docs`).

When changing a potentially niche or subtle part of the language, compiler, or tooling design:

- **Consult `design/` first.** Check whether a relevant decision is already recorded before
  changing behavior, the existing rationale may explain why the current design exists.
- **Keep it in sync.** If a change alters or invalidates a recorded decision, update the
  corresponding file in the same PR.
- **Record new decisions.** If a change introduces a non-obvious design decision that isn't
  captured anywhere, add it to the appropriate file in `design/` (or create a new one).

## Build & Development Commands

The project uses `just` as a task runner and `cargo` for Rust builds. Minimum Rust version: 1.89.0. Run `just --list` to see all available commands.

### Building

```bash
cargo build                          # Build default members (nargo_cli, acvm_cli, etc.)
cargo build -p noirc_frontend        # Build a specific crate

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [noir-lang/noir](https://github.com/noir-lang/noir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
