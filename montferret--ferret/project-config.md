---
trigger: always_on
description: This file is the canonical operating guide for coding agents working in this repository. It is written for Ferret v2 only. If repository documentation conflicts with this file, prefer `Makefile`, `go.mod`, and `.github/workflows/build.yml` for commands, toolchain, and CI behavior.
---

# AGENTS.md

This file is the canonical operating guide for coding agents working in this repository. It is written for Ferret v2 only. If repository documentation conflicts with this file, prefer `Makefile`, `go.mod`, and `.github/workflows/build.yml` for commands, toolchain, and CI behavior.

## Repo snapshot

- Module path: `github.com/MontFerret/ferret/v2`
- Go version: `1.23+`
- Toolchain in `go.mod`: `go1.24.5`
- This repository root is Ferret v2. Do not mix assumptions from the separate v1 branch.
- High-level flow: `Engine` -> `compiler` -> `bytecode.Program` -> `vm.VM`

## Architectural mental model

Ferret v2 is a compiled query language and runtime.

Primary pipeline:

```text
source -> parser -> diagnostics/AST -> compiler -> bytecode.Program -> vm.VM -> runtime values/results
```

Agents should reason about changes by pipeline stage and ownership boundary:
- Source identity, source ranges, and source-origin behavior usually begin in `pkg/source` and affect parser, diagnostics, compiler, formatter, or tooling call sites.
- Syntax changes usually begin in grammar/parser and continue into compiler lowering.
- Diagnostic changes usually involve `pkg/diagnostics` plus the parser/compiler/runtime call sites that create or wrap the diagnostic.
- Semantic/runtime changes usually live in compiler, runtime, or VM.
- Bytecode changes usually require coordinated updates in `pkg/bytecode`, compiler emission, VM execution, and low-level tooling such as `pkg/asm` or debugger metadata.
- Runtime value behavior usually belongs in `pkg/runtime`; VM, stdlib, encoding, and debugger should consume those semantics rather than redefine them.
- Output and materialization changes usually involve `pkg/encoding`, runtime values, and VM result handling.
- Debugging changes usually involve `pkg/debugger`, VM execution hooks/state, compiler or bytecode metadata, and runtime value inspection contracts.
- Built-in module/function changes usually belong in `pkg/stdlib`, while reusable module contracts belong in `pkg/module` or `pkg/sdk`.
- Embedding/API changes usually affect the top-level package and integration boundaries.
- File system access, sandboxing, or path-policy behavior usually belongs in `pkg/fs`, not parser/compiler logic directly.

## Canonical invariants

- Ferret v2 uses a register-based VM.
- `runtime.Value` is the common runtime/VM value abstraction.
- Parser-generated code is derived output, not the source of truth.
- Compiler changes must preserve program semantics expected by the VM.
- Optimizations must preserve correctness before performance.
- Runtime execution errors and internal invariant violations are different classes of failure and should not be collapsed conceptually.
- Do not assume behavior from old design notes or the v1 codebase unless it is reflected in the current v2 code.
- Do not change FQL language semantics as a side effect of refactoring.

## Package map

Agents should begin with the package whose responsibility owns the requested behavior. Do not infer ownership from file names alone when a package in this map already describes the intended boundary.

### Core execution pipeline

* `pkg/source`
    * Owns source text, source identity, source ranges, and source-origin metadata.
    * Prefer this package when behavior depends on where code came from.
    * Parser, diagnostics, compiler, formatter, and tooling may depend on it, but source identity should not be reimplemented in those packages.
* `pkg/parser`
    * Owns FQL syntax parsing, parse-tree processing, parser diagnostics, and parser-generated code integration.
    * Grammar changes should begin under pkg/parser/antlr.
    * pkg/parser/antlr contains grammar sources.
    * pkg/parser/fql contains generated parser and lexer code.
    * Do not hand-edit generated parser artifacts; edit grammar sources and regenerate.
* `pkg/diagnostics`
    * Owns shared diagnostic primitives and formatting support for errors, warnings, spans, labels, notes, hints, and user-facing messages.
    * Parser, compiler, runtime, formatter, and tooling should use shared diagnostic concepts rather than inventing local diagnostic formats.
    * Changes here should preserve diagnostic category, span, label, note, and hint quality.
* `pkg/compiler`
    * Owns semantic analysis, lowering from parsed FQL into bytecode.Program, bytecode emission, and optimization/code generation passes.
    * Compiler changes must preserve the runtime semantics expected by the VM.
    * Do not move runtime-only behavior into the compiler unless the behavior is explicitly compile-time validation or compile-time semantics.
* `pkg/bytecode`
    * Owns the executable program model consumed by the VM and produced by the compiler.
    * Includes instructions, operands, programs, and related executable metadata.
    * Changes here are cross-cutting and usually require coordinated updates in compiler emission, VM execution, debugger metadata, and low-level tooling such as pkg/asm.
* `pkg/vm`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MontFerret/ferret](https://github.com/MontFerret/ferret) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
