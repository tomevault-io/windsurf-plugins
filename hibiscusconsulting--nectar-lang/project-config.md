---
trigger: always_on
description: Nectar exists to prove that the web does not need JavaScript. It is a compiled-to-WASM language where **all logic, all computation, all state management, all rendering decisions run in Rust/WASM**. JavaScript is treated as a thin, unavoidable syscall layer — an impedance mismatch we minimize, not a tool we reach for.
---

# Nectar — Claude Code Instructions

## Core Thesis

Nectar exists to prove that the web does not need JavaScript. It is a compiled-to-WASM language where **all logic, all computation, all state management, all rendering decisions run in Rust/WASM**. JavaScript is treated as a thin, unavoidable syscall layer — an impedance mismatch we minimize, not a tool we reach for.

**NEVER reach for JavaScript.** When implementing any feature, the answer is Rust/WASM. If you think something needs JS, you are almost certainly wrong. The only valid reason for JS is a browser API that WASM physically cannot call (DOM, WebSocket, IndexedDB, fetch, etc.), and even then, the JS function is 1-3 lines with zero logic — a pure bridge.

This is not a guideline. This is the entire point of the language.

**We are building a system for others to use.** Every architectural decision must work at scale, from first principles, without shortcuts. No hardcoded pixel positions when a layout engine should handle it. No JS workarounds when WASM can do it. No "fix it later" — other developers will build on this foundation. The layout engine, the signal system, the rendering pipeline — these are the primitives of a new web platform. They must be correct, fast, and composable. If something takes 450ms when it should take 10ms, that's a bug in the engine, not a reason to bypass it.

## Project Overview

- Users write `.nectar` files; the Rust compiler produces `.wasm` + a single JS syscall file (`core.js`, ~10 KB gzip)
- No garbage collector, no virtual DOM, no JavaScript dependencies, no node_modules
- One Rust binary (`nectar`) handles everything: compile, format, lint, test, dev server, LSP, package management, SSR
- Ownership model inspired by Rust — borrow checking, lifetimes, move semantics
- Declarative UI with fine-grained signals — O(1) updates per binding, no VDOM diffing
- Built-in language keywords for common web patterns: `component`, `store`, `router`, `page`, `form`, `channel`, `auth`, `payment`, `upload`, `db`, `cache`, `agent`, `theme`, `app`
- Standard library auto-included (no imports needed): `crypto`, `format`, `collections`, `BigDecimal`, `url`, `search`, `debounce`, `throttle`, `pagination`, `toast`, `skeleton`, `mask`, `chart`, `csv`, `datepicker`
- **Repo**: `git@github.com:HibiscusConsulting/nectar-lang.git`
- **License**: MIT
- **Owner**: Blake Burnette (jbburnette2@gmail.com) / Hibiscus Consulting

## Build & Test

```bash
# Build the compiler
cargo build                          # from /compiler or workspace root

# Run all tests
cargo test                           # 2547 tests

# Run tests for a specific module
cargo test --lib parser              # just parser tests
cargo test --lib codegen             # just codegen tests

# Test coverage (requires cargo-tarpaulin)
cargo tarpaulin --out json --output-dir .
```

The binary is `nectar` (defined in `compiler/src/main.rs`). CLI commands:

```bash
nectar build app.nectar --emit-wasm  # Compile to WebAssembly
nectar dev --src . --port 3000       # Dev server with hot reload
nectar fmt app.nectar                # Format
nectar lint app.nectar               # Lint
nectar test app.nectar               # Test
nectar check app.nectar              # Type-check + borrow-check without building
nectar build --ssr                   # Server-side rendering
nectar install                       # Package manager
nectar --lsp                         # LSP server
```

## Architecture

```
nectar-lang/
├── compiler/src/              # Rust compiler — single binary, 105K+ lines
│   ├── main.rs                # CLI entry point (clap) (~2650 lines)
│   ├── lexer.rs               # Tokenizer (~1140 lines)
│   ├── token.rs               # Token types (~300 lines)
│   ├── parser.rs              # Parser → AST (~10900 lines)
│   ├── ast.rs                 # AST node types (~1440 lines)
│   ├── type_checker.rs        # Type checking (~9260 lines)
│   ├── borrow_checker.rs      # Ownership/borrowing rules (~5250 lines)
│   ├── codegen.rs             # AST → WAT (~37K lines, largest file)
│   ├── wasm_binary.rs         # WAT → .wasm binary (~3270 lines)
│   ├── wasm_opt.rs            # WASM optimization passes (~480 lines)
│   ├── const_fold.rs          # Constant folding (~1740 lines)
│   ├── dce.rs                 # Dead code elimination (~1490 lines)
│   ├── tree_shake.rs          # Tree shaking (~1940 lines)
│   ├── exhaustiveness.rs      # Pattern match exhaustiveness checking (~1850 lines)
│   ├── monomorphize.rs        # Generic function monomorphization (~870 lines)
│   ├── contract_infer.rs      # Contract shape inference from fetch responses (~980 lines)
│   ├── contract_verify.rs     # Compile-time contract validation (~580 lines)
│   ├── formatter.rs           # nectar fmt (~4450 lines)
│   ├── linter.rs              # nectar lint (~3850 lines)
│   ├── lsp.rs                 # Language server protocol with dot-completion (~1680 lines)
│   ├── ssr.rs                 # Server-side rendering (~1880 lines)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HibiscusConsulting/nectar-lang](https://github.com/HibiscusConsulting/nectar-lang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
