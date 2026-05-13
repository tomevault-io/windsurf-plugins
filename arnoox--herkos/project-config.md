---
trigger: always_on
description: `herkos` is a compilation pipeline that transpiles WebAssembly modules into memory-safe Rust code with compile-time isolation guarantees. The goal is to replace runtime hardware-based memory protection (MMU/MPU) with type-system-enforced safety.
---

# CLAUDE.md — herkos

## Project overview

`herkos` is a compilation pipeline that transpiles WebAssembly modules into memory-safe Rust code with compile-time isolation guarantees. The goal is to replace runtime hardware-based memory protection (MMU/MPU) with type-system-enforced safety.

The pipeline: **WebAssembly → Rust source → Safe binary**

## Documentation

| Document | Purpose |
|----------|---------|
| `docs/REQUIREMENTS.md` | What the system must do — formal requirements with REQ_* IDs |
| `docs/SPECIFICATION.md` | How it works — module representation, architecture, transpilation rules, integration, performance. Also includes getting started guide. |
| `docs/FUTURE.md` | Planned but unimplemented features — verified/hybrid backends, temporal isolation, contract-based verification |

## Repository structure

The project is a Rust workspace with three crates:

| Crate | Purpose | `no_std` |
|-------|---------|----------|
| `crates/herkos/` | CLI transpiler: parses `.wasm` binaries, emits Rust source code | No (`std`) |
| `crates/herkos-runtime/` | Runtime library shipped with transpiled output | **Yes** |
| `crates/herkos-tests/` | Integration tests + benchmarks: WAT/C/Rust → .wasm → transpile → test | No (`std`) |

### Transpiler pipeline (`crates/herkos/src/`)

```
.wasm → parser/ → ir/builder/ → optimizer/ → backend/safe.rs → codegen/ → rustfmt
        (wasmparser)  (SSA IR)    (dead blocks)  (SafeBackend)   (Rust source)
```

Key modules:
- `parser/` — Wasm binary parsing via `wasmparser` crate
- `ir/` — SSA-form intermediate representation (`ModuleInfo`, `IrFunction`, `IrBlock`, `IrInstr`)
  - `ir/builder/` — Wasm → IR translation (core.rs, translate.rs, assembly.rs, analysis.rs)
- `optimizer/` — IR optimization passes (currently: dead block elimination)
- `backend/` — Backend trait + `SafeBackend` (bounds-checked, no unsafe)
- `codegen/` — IR → Rust source (module.rs, function.rs, instruction.rs, traits.rs, export.rs, constructor.rs)

### Runtime (`crates/herkos-runtime/src/`)

- `memory.rs` — `IsolatedMemory<MAX_PAGES>`: load/store methods, memory.grow/size, Kani proofs
- `table.rs` — `Table<MAX_SIZE>`, `FuncRef`: indirect call dispatch
- `module.rs` — `Module<G, MAX_PAGES, TABLE_SIZE>`, `LibraryModule<G, TABLE_SIZE>`
- `ops.rs` — Wasm arithmetic operations with trap handling (div, rem, trunc)
- `lib.rs` — `WasmTrap`, `WasmResult<T>`, `ConstructionError`, `PAGE_SIZE`

### Tests (`crates/herkos-tests/`)

- `build.rs` — Compiles WAT/C/Rust sources to `.wasm`, invokes transpiler, writes to `OUT_DIR`
- `tests/` — Integration tests: arithmetic, memory, control flow, imports/exports, E2E (C and Rust)
- `benches/` — Criterion benchmarks (Fibonacci)
- `data/rust/` — Pre-generated Rust test modules

## Build and test

```bash
cargo build                    # build all crates
cargo test                     # run all tests
cargo clippy --all-targets     # lint (CI enforced)
cargo fmt --check              # format check (CI enforced)
cargo bench -p herkos-tests    # benchmarks
```

Run a single crate's tests:
```bash
cargo test -p herkos
cargo test -p herkos-runtime
cargo test -p herkos-tests
```

CLI usage:
```bash
cargo run -p herkos -- input.wasm --output output.rs
```

## Key architectural concepts

### Memory model

Wasm linear memory is `IsolatedMemory<const MAX_PAGES: usize>` — a 2D array `[[u8; PAGE_SIZE]; MAX_PAGES]` with `active_pages` tracking. Fully allocated at compile time, no heap. See `crates/herkos-runtime/src/memory.rs` and SPECIFICATION.md §2.1.

### Module types

Two kinds:
1. **`Module<G, MAX_PAGES, TABLE_SIZE>`** — Owns memory (process-like)
2. **`LibraryModule<G, TABLE_SIZE>`** — Borrows caller's memory (library-like)

Each has a **Globals struct** `G` (one typed field per mutable Wasm global) and a **Table** for indirect calls. See `crates/herkos-runtime/src/module.rs` and SPECIFICATION.md §2.2.

### Capability-based security via traits

- **Imports** → trait bounds on generic host parameter `H`
- **Exports** → trait implementations on the module struct
- **Zero-cost**: monomorphization, no vtables, no trait objects in hot paths
- **WASI**: standard traits (`WasiFd`, `WasiPath`, `WasiClock`, `WasiRandom`) shipped with runtime

See SPECIFICATION.md §2.4–2.6.

### Function calls

- **Direct** (`call`): regular Rust function calls with state threaded through
- **Indirect** (`call_indirect`): safe static match dispatch over `func_index`, no function pointers
- **Structural type equivalence**: canonical type index mapping at transpile time

See SPECIFICATION.md §4.5.

### Error handling

- `WasmTrap` enum with 7 variants (OutOfBounds, DivisionByZero, IntegerOverflow, Unreachable, IndirectCallTypeMismatch, TableOutOfBounds, UndefinedElement)
- `WasmResult<T> = Result<T, WasmTrap>` — no panics, no unwinding
- `ConstructionError` for programming errors during module instantiation

### Current status

- **Implemented**: Safe backend only (runtime bounds checking, no unsafe in output)
- **Not yet implemented**: Verified backend, hybrid backend, `--max-pages` CLI effect, WASI traits

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arnoox/herkos](https://github.com/arnoox/herkos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
