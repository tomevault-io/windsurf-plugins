---
trigger: always_on
description: Celox is a JIT simulator for Veryl HDL. It compiles Veryl designs with Cranelift for high-speed simulation. Future plans include SystemVerilog/Verilog support.
---

# CLAUDE.md

## Project Overview

Celox is a JIT simulator for Veryl HDL. It compiles Veryl designs with Cranelift for high-speed simulation. Future plans include SystemVerilog/Verilog support.

## Build Commands

```bash
cargo build              # Build all crates
cargo test               # Run all tests
cargo test -p celox      # Run tests for the core crate only
cargo run -p celox-ts-gen -- --help  # TypeScript type generator CLI
```

### Snapshot Tests

```bash
cargo insta test         # Run snapshot tests
cargo insta accept       # Accept snapshot changes
```

## Workspace Structure

| Crate / Package | Description |
|---|---|
| `crates/celox` | Core simulator (IR, JIT compilation, runtime) |
| `crates/celox-macros` | Procedural macros |
| `crates/celox-napi` | N-API bindings for Node.js |
| `crates/celox-ts-gen` | CLI tool for TypeScript type generation |
| `crates/celox-bench-sv` | SystemVerilog generator for Verilator benchmarks |
| `packages/celox` | TypeScript runtime package |
| `packages/vite-plugin` | Vite plugin |

## Veryl Submodule

The `deps/veryl/` directory contains a fork of Veryl (`tignear/veryl`). The workspace depends on `veryl-analyzer`, `veryl-emitter`, `veryl-parser`, `veryl-metadata`, and `veryl-path` from this submodule.

- `default-features = false` is set on `veryl-parser` to suppress parser regeneration during builds.
- After updating the submodule, run `cargo test` to verify compatibility.

### Analyzer API

The Veryl analyzer pass functions (`analyze_pass1`, `analyze_post_pass1`, `analyze_pass2`, `analyze_post_pass2`) return `Vec<AnalyzerError>`. All 4 passes must be called and their errors checked. `SimulatorError::Analyzer` wraps these errors.

### Writing Veryl in Tests

The analyzer enforces strict checks. When writing Veryl source in integration tests:

- **Clock domain annotations**: Multi-clock designs require `'a`/`'b` (or `'_` for single-clock) on all ports and vars. Cross-domain access needs `unsafe (cdc) { ... }`.
- **Logical operators on multi-bit**: `a && b` / `a || b` / `!a` are rejected for operands wider than 1 bit. Use reduction: `(|a) && (|b)`, `!(|a)`.
- **logic → bit assignment**: Requires explicit cast `as u8`.
- **SV keywords as identifiers**: Forbidden (e.g. `reg`). Use alternatives like `r_val`.
- **Clock from logic**: A `var` of type `logic` cannot be used as a clock. Use an external `clock` input or `let gated: '_ clock = clk_input & en;` (first operand must be clock-typed).
- **Self-referential assign**: `assign v = f(v);` is rejected as `UnassignVariable`. Use `always_comb` with `if`/`else` branches if possible, or redesign the circuit.

## Optimizer Options

GCC-style optimization level presets with per-pass overrides.

### OptLevel

`OptLevel` (`crates/celox/src/optimizer.rs`) sets defaults for SIR passes, Cranelift backend, and DSE:

| Level | SIR Passes | DSE | Cranelift |
|---|---|---|---|
| `O0` | TailCallSplit only | Off | `fast_compile()` |
| `O1` (default) | All 18 passes | Off | Speed / Backtracking |
| `O2` | All 18 passes | PreserveTopPorts | Speed / Backtracking |

### SirPass

`SirPass` enum (`crates/celox/src/optimizer.rs`) — all 18 SIR optimization passes:

| Pass | Description |
|---|---|
| `StoreLoadForwarding` | Propagates stored values to subsequent loads |
| `HoistCommonBranchLoads` | Hoists loads shared across all branches to the entry |
| `BitExtractPeephole` | Converts `(value >> shift) & mask` → direct ranged loads |
| `OptimizeBlocks` | General block-level optimizations (dead block removal, merging) |
| `SplitWideCommits` | Splits wide commit operations into narrower ones |
| `CommitSinking` | Sinks commit operations closer to their use site |
| `InlineCommitForwarding` | Inlines values forwarded through commit operations |
| `EliminateDeadWorkingStores` | Removes working-memory stores that are never read |
| `Reschedule` | Reorders instructions for better backend codegen |
| `CoalesceStores` | Merges consecutive narrow stores into wider Concat+Store |
| `Gvn` | Global value numbering / dead code elimination |
| `ConcatFolding` | Folds redundant Concat operations |
| `XorChainFolding` | Folds XOR chains |
| `VectorizeConcat` | Vectorizes Concat patterns in combinational blocks |
| `SplitCoalescedStores` | Splits wide coalesced stores back after reschedule |
| `PartialForward` | Partial store-load forwarding in combinational blocks |
| `IdentityStoreBypass` | Detects identity copies and registers address aliases |
| `TailCallSplit` | Splits large functions into tail-call chains (enabled even at O0) |

### Cranelift Backend Optimization

`CraneliftOptions` (`crates/celox/src/optimizer.rs`) provides fine-grained Cranelift backend control:

| Field | Type | Default | Description |
|---|---|---|---|
| `opt_level` | `CraneliftOptLevel` | `Speed` | Optimization level (`None` / `Speed` / `SpeedAndSize`) |
| `regalloc_algorithm` | `RegallocAlgorithm` | `Backtracking` | Register allocator (`Backtracking` = better code / `SinglePass` = faster compile) |
| `enable_alias_analysis` | `bool` | `true` | Alias analysis in egraph pass (only effective when `opt_level` ≠ `None`) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [celox-sim/celox](https://github.com/celox-sim/celox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
