---
trigger: always_on
description: A PHP-to-native compiler written in Rust. Compiles a static subset of PHP to ARM64 assembly, producing standalone macOS Mach-O binaries. No interpreter, no VM, no runtime dependencies.
---

# elephc — Developer Guide

## What is this

A PHP-to-native compiler written in Rust. Compiles a static subset of PHP to ARM64 assembly, producing standalone macOS Mach-O binaries. No interpreter, no VM, no runtime dependencies.

## Build & run

```bash
cargo build              # dev build
cargo build --release    # optimized build
cargo run -- file.php    # compile a PHP file
```

The compiler outputs a native binary next to the source file (e.g., `file.php` → `file`).

## Test policy

**Every feature must have tests before it's considered done.** The test suite is the primary quality gate.

### Running tests

```bash
cargo test                          # run all tests (slow — ~5-6 min due to as+ld per codegen test)
cargo test -- --include-ignored     # run ALL tests including those requiring external libs
cargo test --test codegen_tests     # run only end-to-end tests
cargo test test_fizzbuzz            # run a specific test
```

Some tests are marked `#[ignore]` because they require external libraries (e.g., SDL2) not available in CI. **Before committing, always run `cargo test -- --include-ignored` locally** to verify nothing is broken — including ignored tests.

### Test strategy during development

The full test suite is slow because each codegen test spawns `as` + `ld` + runs the binary. To avoid waiting several minutes on every change:

1. **While developing a feature**: run only the tests for that feature (`cargo test test_my_feature`)
2. **When the feature is complete**: run the full suite once (`cargo test`) to check for regressions
3. **PHP cross-check**: opt-in via `ELEPHC_PHP_CHECK=1 cargo test` — verifies output matches PHP interpreter

### Test structure

| File | What it tests | How |
|---|---|---|
| `tests/lexer_tests.rs` | Tokenization | Asserts token sequences from source strings |
| `tests/parser_tests.rs` | AST construction | Asserts AST node structure and operator precedence |
| `tests/codegen_tests.rs` | Full pipeline (end-to-end) | Compiles PHP → binary, runs it, asserts stdout |
| `tests/error_tests.rs` | Error reporting | Asserts that invalid programs produce the right error messages |

### Test coverage requirements

- **New language construct** (keyword, operator, statement): needs lexer, parser, codegen, AND error tests
- **New operator**: needs a Pratt parser binding power test verifying precedence relative to adjacent operators
- **New statement type**: needs at least one codegen test showing correct output, one test for edge cases (empty body, nested), and one error test for malformed syntax
- **New built-in function**: needs codegen tests for normal use and error test for wrong argument count/types
- **Bug fix**: must include a regression test that would have caught the bug
- **Every feature also needs an example** in `examples/`. If an existing example can showcase the new feature naturally, update it. Otherwise, create a new `examples/<name>/main.php` with its own `.gitignore` (containing `*.s`, `*.o`, `main`). Examples should be small, readable programs that demonstrate real use cases — not just test cases.

### Writing codegen tests

Codegen tests compile inline PHP source and assert stdout:

```rust
#[test]
fn test_my_feature() {
    let out = compile_and_run("<?php echo 1 + 2;");
    assert_eq!(out, "3");
}
```

Each test runs in an isolated temp directory. Tests run in parallel — the `compile_and_run` helper handles isolation automatically.

## Architecture

```
PHP source → Lexer (tokens) → Parser (AST) → Resolver (include/require) → NameResolver (namespace/use/FQN canonicalization) → Type Checker → Codegen (ARM64 asm) → as + ld → binary
```

### Key modules

| Module | Entry point | Responsibility |
|---|---|---|
| `src/lexer/` | `tokenize()` | Source → `Vec<(Token, Span)>` |
| `src/parser/` | `parse()` | Tokens → `Program` (Vec of Stmt). Pratt parser for expressions |
| `src/resolver.rs` | `resolve()` | Resolves `include`/`require` by inlining referenced files. Runs before namespace/name canonicalization |
| `src/name_resolver.rs` | `resolve()` | Applies namespace/use rules, rewrites references to canonical fully-qualified names, and flattens namespace-only AST nodes before type checking |
| `src/types/` | `check()` | Type checking, returns `CheckResult` with `TypeEnv`, function/class/interface/FFI metadata, and the internal `Mixed` type for heterogeneous assoc-array values |
| `src/codegen/` | `generate()` | AST → ARM64 assembly string. Top-level orchestration lives in `mod.rs`, while most lowering lives under `expr/`, `stmt/`, and `runtime/` |
| `src/errors/` | `report()` | Error formatting with line:col |
| `src/span.rs` | `Span` | Source position (line, col) attached to all AST nodes |

### Codegen layout

- `src/codegen/expr.rs` is mainly a dispatcher; most expression lowering now lives in focused helpers under `src/codegen/expr/`
- `src/codegen/stmt.rs` is mainly a dispatcher; most statement lowering now lives in focused helpers under `src/codegen/stmt/`
- `src/codegen/runtime/mod.rs` emits runtime code (`__rt_*` routines)
- `src/codegen/runtime/data.rs` emits runtime `.data` / `.bss` symbols and metadata tables

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [illegalstudio/elephc](https://github.com/illegalstudio/elephc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
