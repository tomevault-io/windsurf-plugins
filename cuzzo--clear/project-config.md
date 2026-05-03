---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## MiniVM Rules

The active MiniVM path is `bc_emitter.rb` + `_bc_runner.cht` (bytecode compiler + interpreter).

`scheme_transpiler.rb` and `interpreter.cht` are **deprecated**. Do not add features to them.

**NEVER parse Zig code strings in the MiniVM to generate bytecode. Not one character of Zig.**
`MIR::InlineZig` and `MIR::RawZig` are Zig backend artifacts. When the bc_emitter encounters
them, it must use the AST fallback (`compile_ast_stmt` / `compile_ast_expr`), never inspect
the `.code` string. If no AST node is available, raise `Unimplemented`.

## Project Overview

**CLEAR** is a memory-safe programming language that combines the ease of Ruby/Python with Rust-like safety. It features arena-based memory management (no garbage collector), ownership semantics, and separates **Types** from **Capabilities**.

## Build & Test Commands

```bash
# --- clear CLI (preferred) ---
./clear build foo.cht                # Default: Zig backend, ~2s, safety checks, 64KB stacks
./clear build foo.cht -o bin/app     # Custom output path
./clear build foo.cht --stack-check  # Build + verify stack usage per function via objdump
./clear build foo.cht --optimized    # LLVM backend, -O ReleaseFast (~22s, 16KB stacks)
./clear build foo.cht --safe         # LLVM backend, -O ReleaseSafe (~28s, safety + optimization)
./clear run foo.cht                  # Build + execute
./clear run foo.cht -- --port 8080   # Pass args to program
./clear test foo.cht                 # Test single file with leak detection
./clear test transpile-tests/        # Test all .cht files in directory (130 tests)
./clear profile foo.cht              # Build + run with heap/CPU profiling
./clear doctor foo.profile/          # Analyze profile data, print optimization advice

# --- Full test suites ---
bundle install                       # Install Ruby dependencies
bundle exec prspec spec/              # Run all Ruby specs in parallel (~1s, excludes integration)
bundle exec prspec spec/ --tag integration  # Run integration tests (builds binaries, ~3-4 min)

# Package integration test
cd transpile-tests/module-integration && zig build test

# FFI integration test
cd transpile-tests/ffi-integration && zig build test

# Example tests (run before committing)
./clear test examples/testing/basic_test.cht
./clear test examples/testing/stub_ufcs.cht
```

### Build Modes

| Flag | Backend | Time | Safety | Stacks | Use |
|------|---------|------|--------|--------|-----|
| (default) | Zig x86 | ~2s | Bounds/overflow | 64KB | Development |
| `--optimized` | LLVM | ~22s | None | 16KB | Benchmarks, deployment |
| `--safe` | LLVM | ~28s | Bounds/overflow | 16KB | Debugging optimized builds |

**NOTE**: The default build does NOT have stack-smash protection (`__morestack`). That
requires the LLVM backend with the custom machine pass (not yet integrated into `clear`).
Zig's safety checks (bounds, overflow, null) ARE enabled in the default build. The 64KB
fiber stacks compensate for the larger stack frames that safety instrumentation produces.

### Test Suites

Run **all four** after making changes to the compiler:
- **Ruby unit specs**: `bundle exec prspec spec/` (parallel, ~1s, excludes integration)
- **transpile-tests**: `./clear test transpile-tests/` (130 tests)
- **module-integration**: `cd transpile-tests/module-integration && zig build test`
- **ffi-integration**: `cd transpile-tests/ffi-integration && zig build test`

Run **integration specs** after changes to the CLI or stack verifier:
- **Ruby integration specs**: `bundle exec prspec spec/ --tag integration` (~3-4 min, builds binaries)

## Benchmarks

```bash
# Benchmark runner modes
ruby benchmarks/runner.rb --smoke benchmarks/server/02_json_api/      # CLEAR only, fast (~5s)
ruby benchmarks/runner.rb --fast benchmarks/sequential/04_hashmap/    # All langs, reduced (~30s)
ruby benchmarks/runner.rb benchmarks/sequential/04_hashmap/           # Normal (default)
ruby benchmarks/runner.rb --release benchmarks/sequential/04_hashmap/ # Exhaustive (5x load)
ruby benchmarks/runner.rb --sequential                                 # Sequential benchmarks
ruby benchmarks/runner.rb --concurrent                                 # Concurrent benchmarks
ruby benchmarks/runner.rb --server                                     # Server benchmarks
ruby benchmarks/runner.rb --all                                        # All benchmarks
ruby benchmarks/runner.rb --smoke --all                                # Smoke test all benchmarks
ruby benchmarks/runner.rb --cores=2 benchmarks/concurrent/09_kvstore/ # Control core count
```

See `benchmarks/README.md` for the full benchmark index and details.

## Code Quality / Coverage Reports

Three measurements: line + branch coverage (SimpleCov), per-method
complexity (Flog), code smells (Reek), and structural duplication
(Flay) -- all aggregated by RubyCritic. Run after meaningful
changes to track tech-debt drift.

```bash
# 1. Spec coverage. SimpleCov is auto-wired via spec/spec_helper.rb;
#    parallel_rspec workers each write a unique resultset entry.
bundle exec prspec spec/

# 2. Integration coverage. transpile-tests/gen.rb and the `clear` CLI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cuzzo/clear](https://github.com/cuzzo/clear) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
