---
trigger: always_on
description: <!-- agent-pmo:74cf183 -->
---

# CLAUDE.md
<!-- agent-pmo:74cf183 -->

This file provides guidance for agents when working with code in this repository.

⚠️ NEVER ASk THE USER QUESTIONS! USER YOUR JUDGEMENT. ACT AUTONOMOUSLY ⚠️
⚠️ **NEVER DUPLICATE CODE** - Edit in place, never create new versions. Use deslop - find-similar before adding code, and deslop-top-offenders after modifying code ⚠️
⚠️ DO NOT USE GIT - ESPECIALLY NOT PUTTING YOUR SIGNATURE ON COMMITS ⚠️
⚠️ PRACTICE TOKEN ECONOMICS ⚠️
⚠️ ZERO DUPLICATE CODE ⚠️

## Core Development Principles

- **NO PLACEHOLDERS** - Fix existing placeholders or fail with error
- **RUN make ci ROUTINELY** - Many clippy lints can be easily fixed with auto fix. Don't try to fix them yourself
- **PREFER EXPANDING EXISTING EXAMPLES AND TESTS** - Don't add new examples/tests
- **DO NOT USE GIT - IN PARTICULAR, DO NOT STAMP YOURSELF AS COAUTHOR** - unless explicitly requested
- **MAKE EXAMPLES (TESTS) CONCISE AND MIX WITH MANY LANGUAGE CONSTRUCTS** - Don't create many files with overlapping functionality
- **KEEP ALL FILES UNDER 500 LOC** - Break large files into focused modules 
- **KEEP FUNCTIONS BELOW 20 LOC**
- **FP STYLE CODE** - pure functions over OOP style
- **Handle all panics and return Result<T,E>** instead of throwing
- **USE CONSTANTS** - Name values meaningfully instead of using literals

## Rust

- **Panics are illegal** Return Result<T,E>
- **unwrap() and similar are illegal** Use pattern matching

## Osprey

- **Osprey is an FP language** - Use constructs that other FP languages use:
  - Immutable types
  - Expressions over statements
  - Avoid brackets where they are not necessary (ML style)
  - Use algebraic effects for abstractions
  - The best function is a single expression with no side effects (pure)
  - Avoid consecutive statements and assignments, even when assignments add
    clarity
- **LEAN ON TYPE INFERENCE — DO NOT WRITE REDUNDANT TYPE ANNOTATIONS** -
  Osprey is Hindley-Milner: every type the compiler can infer must be left
  off. This is a core style rule of the language — less redundancy.
  - **Never annotate function parameters** when their type is inferable from
    the body or call site. Write `fn add(a, b) = a + b`, NOT
    `fn add(a: int, b: int) = a + b`.
  - **Never annotate a function return type** when it is inferable. Write
    `fn isEven(x) = (x % 2) == 0`, NOT `fn isEven(x: int) -> bool = ...`.
  - **Never annotate lambda parameters** when inferable: `|x| => x * 2`, not
    `|x: int| => x * 2`.
  - Keep an annotation ONLY when the compiler genuinely cannot infer it: an
    empty literal with no context (`let xs: List<int> = []`), an `extern` /
    ambiguous return, an unconstrained polymorphic type variable, or a
    load-bearing return type that forces `Result<T, MathError>` to
    auto-unwrap to `T`. If removing an annotation still compiles and produces
    identical output, it was redundant — remove it.
  - This applies to ALL `.osp` you write or touch — `examples/tested/`,
    `benchmarks/`, docs, and website snippets alike.
- **NO CONSECUTIVE PRINT CALLS IN OSP** - Use string interpolation! Consolidate consecutive prints into singular interpolated strings!!!

## Commands

**Primary Development Commands (run from the repo root):**
```bash
make build         # C runtime archives + cargo build --release + VSCode extension
make test          # All tests + coverage thresholds + differential example harness
make lint          # cargo clippy + extension lint
make fmt           # Format all code in-place (CHECK=1 for read-only check)
make ci            # lint + test + build (full CI simulation)
make clean         # Clean all build artifacts
```

**Development Commands:**
```bash
make run FILE=<path>       # Compile and run an Osprey file (osprey <file> --run)
make install               # Install osprey + runtime archives system-wide
make _rebuild-install-vsix  # Rebuild + reinstall the VSCode extension (macOS)
```

The compiler binary lands at `target/release/osprey`.

**VSCode Extension:**
```bash
cd vscode-extension
npm install && npm run compile    # Build VSCode extension
npm test                         # Run extension tests
```

**Website Development:**
```bash
cd website
npm install && npm run dev       # Start local development server
npm run build                    # Build static site
```

**CSS HARD BUDGET 1.8K LOC** BLOGS, SPECS, DOCS = PROSE = SAME CSS NAME PREFIX PROSE
**ZERO TAILWIND** CONVERT TO CSS IMMEDIATELY

**WebCompiler (Browser-based):**
```bash
cd webcompiler  
npm install && npm start         # Start web-based compiler service
```

## High-Level Architecture

**Repository Structure:**
- `crates/` - Core Osprey compiler (Rust workspace → LLVM)
- `tree-sitter-osprey/` - Tree-sitter grammar for parsing
- `compiler/` - Pure-C runtime sources (`runtime/`) + example programs (`examples/`)
- `vscode-extension/` - VSCode language support with TypeScript
- `website/` - Documentation site using 11ty static site generator
- `webcompiler/` - Node.js web service for browser compilation
- `homebrew-package/` - Homebrew tap for macOS installation

**Compiler Architecture (Rust-based):**
- **Parser**: Tree-sitter grammar (`tree-sitter-osprey/`) consumed by `crates/osprey-syntax`
- **AST**: Abstract Syntax Tree types in `crates/osprey-ast`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nimblesite/osprey](https://github.com/Nimblesite/osprey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
