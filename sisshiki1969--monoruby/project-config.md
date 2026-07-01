---
trigger: always_on
description: A comprehensive guide for AI assistants working on this repository.
---

# CLAUDE.md — monoruby

A comprehensive guide for AI assistants working on this repository.

## Project Overview

**monoruby** is a Ruby implementation written from scratch in Rust, featuring a register-based bytecode VM and a just-in-time (JIT) compiler. It is performance-focused and is comparable to ruby 3.4+YJIT on the optcarrot benchmark.

- **Parser**: Ruby source is parsed by **prism** (the official Ruby parser, consumed as the `ruby-prism` crate); monoruby converts prism's tree into its own AST (`monoruby/src/ast/`, `monoruby/src/parser/`). The old hand-written `ruruby-parse` crate has been removed.
- **Platform**: x86-64 **and** aarch64. The VM-tier backend (bytecode VM, invokers, native-function wrappers) and the JIT emit machine code directly via `monoasm`, with a per-`target_arch` backend under `codegen/arch/`. Both backends lower the **full** AsmInst set to machine code; aarch64 materializes large frame/field/sp offsets through scratch registers rather than bailing, so it never declines a compile (`compile_asmir`'s `bool` return is vestigial — see `doc/arch_difference.md`). Tested on Linux/x86-64 and macOS Apple Silicon (arm64-apple-darwin).
- **Rust channel**: Nightly (`nightly-2026-05-18`, pinned in `rust-toolchain.toml`)
- **No dependency on CRuby** or any other Ruby runtime

---

## Repository Layout

```
monoruby/                   # Workspace root
├── monoruby/               # Main crate — the Ruby interpreter & JIT
│   ├── src/
│   │   ├── main.rs         # CLI entry point (monoruby binary)
│   │   ├── lib.rs          # Library root; re-exports public API
│   │   ├── parser/         # prism → monoruby-AST bridge
│   │   │   ├── mod.rs
│   │   │   └── prism_backend.rs # Drives the `ruby-prism` crate
│   │   ├── ast/            # monoruby AST (node, lvar_collector, source_info, error)
│   │   ├── alloc.rs        # Custom GC allocator (mark-and-sweep)
│   │   ├── id_table.rs     # Interned identifier table (IdentId)
│   │   ├── value.rs        # Value type (tagged-union, 64-bit, NonZeroU64)
│   │   ├── value/
│   │   │   ├── numeric.rs  # Numeric helpers (Fixnum/Float/BigInt)
│   │   │   └── rvalue/     # Heap-allocated Ruby values (RValue)
│   │   ├── bytecode.rs     # Bytecode instruction index types (BcIndex)
│   │   ├── bytecodegen/    # AST → register-based bytecode compiler
│   │   │   ├── bytecodegen.rs
│   │   │   ├── inst.rs     # Bytecode instruction definitions
│   │   │   ├── expression.rs
│   │   │   ├── statement.rs
│   │   │   └── method_call/
│   │   ├── executor/       # Bytecode interpreter (VM)
│   │   │   ├── executor.rs
│   │   │   ├── frame.rs    # Stack/control frame layout
│   │   │   ├── op/         # Operator dispatch
│   │   │   ├── inline.rs   # Inline method dispatch table
│   │   │   └── constants.rs
│   │   ├── codegen/        # JIT compiler + arch-neutral codegen glue
│   │   │   ├── codegen.rs  # Thread-local CODEGEN singleton; arch-neutral types
│   │   │   ├── compiler.rs # JIT compilation entry point
│   │   │   ├── jit_module.rs # Arch-neutral: handle_error, ErrorReturn, …
│   │   │   ├── arch.rs     # target_arch switch (x86_64 / aarch64)
│   │   │   ├── arch/       # Per-arch backends (VM-tier + JIT emission), mirrored
│   │   │   │   ├── x86_64/ # {codegen,jit_module,invoker,wrapper,vmgen(+vmgen/),
│   │   │   │   │           #  compile(+compile/),guard}  ← JIT lowering: compile,guard
│   │   │   │   └── aarch64/# {codegen,jit_module,invoker,wrapper,vmgen,compile,guard}
│   │   │   ├── runtime/    # JIT runtime helpers
│   │   │   └── jitgen/     # Bytecode → TraceIR → AsmIR (arch-neutral front-end)
│   │   │       ├── trace_ir.rs
│   │   │       ├── state/  # Abstract interpreter state
│   │   │       ├── asmir/  # AsmIR defs + arch-neutral lowering dispatch (compile_shared)
│   │   │       └── compile.rs # TraceIR → AsmIR (the per-arch AsmIR→machine-code
│   │   │                      # backend lives in arch/<arch>/compile, guard)
│   │   ├── globals/        # Global interpreter state
│   │   │   ├── globals.rs  # Globals struct (main_object, Store, …)
│   │   │   ├── store/      # Function, class, and call-site tables
│   │   │   ├── error.rs    # MonorubyErr type
│   │   │   ├── method.rs   # Method lookup helpers
│   │   │   └── require.rs  # File loading (`require`/`load`)
│   │   ├── builtins/       # Built-in Ruby class implementations
│   │   │   ├── builtins.rs # init_builtins() — registers all classes
│   │   │   ├── array.rs, string.rs, hash.rs, numeric/, …
│   │   │   └── …
│   │   └── tests.rs        # Test harness helpers
│   └── build.rs            # Build script (sets up library path file)
│
├── monoruby_attr/          # Proc-macro crate (#[monoruby_builtin], etc.)
├── rubymap/                # Order-preserving hash map for Ruby Hash
├── hashbrown/              # Vendored hashbrown (local fork)
├── ruby_traits/            # Shared trait definitions
│
├── benchmark/              # Ruby benchmark scripts + YAML configs
├── bin/                    # Shell helper scripts
│   ├── test                # Full test + coverage run (used in CI)
│   ├── test-aarch64        # aarch64 test run (qemu / cross)
│   ├── setup-aarch64-cross # Set up the aarch64 cross-build toolchain
│   ├── bench               # Benchmark runner

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sisshiki1969/monoruby](https://github.com/sisshiki1969/monoruby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
