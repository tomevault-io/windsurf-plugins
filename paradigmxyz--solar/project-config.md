---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

## Project Overview

Solar is a blazingly fast, modular Solidity compiler written in Rust, aiming to be a modern alternative to solc.

For testing and comparing behavior and semantics, the current tracked solc version (usually the latest stable release) is always available as a submodule `./testdata/solidity`.

## Commands

```bash
cargo build                            # Build
cargo nextest run --workspace          # Run tests (faster than cargo test)
cargo llvm-cov nextest --workspace     # Test coverage
cargo uitest                           # Run UI tests
cargo uibless                          # Update UI test expectations
cargo fmt --all                        # Format
cargo clippy --workspace --all-targets # Lint
cargo run -- file.sol                  # Run compiler
cargo run -- -Zhelp                    # Unstable flags help
```

DO NOT USE `cargo test` DIRECTLY IF YOU CAN AVOID IT.

NEVER RUN TESTS WITH `--all-features`. This enables "tracy" which has heavy overhead per-process, which the UI tests spawn lots of, increasing test times to minutes and 100% CPU for no reason.

## Architecture

- **solar-parse**: Lexer and parser
- **solar-ast**: AST definitions and visitors
- **solar-sema**: Semantic analysis (symbol resolution, type checking)
- **solar-codegen**: MIR construction, MIR optimizations, and EVM backend codegen
- **solar-interface**: Diagnostics and source management
- **solar-cli**: Command-line interface

Pipeline: Lexing -> Parsing -> Semantic Analysis -> MIR -> EVM backend -> bytecode

### MIR and EVM IR

- **MIR** is the compiler's higher-level codegen IR. It is typed, function-based,
  and is the right place for Solidity-aware and SSA-style optimizations such as
  mem2reg/frame-slot promotion, inlining, CSE/GVN/PRE, SCCP, LICM, and loop
  analysis.
- **EVM IR** is the lower, Machine-IR-like backend layer. It comes after
  function calls and virtual values have been lowered away. It models asm-like
  basic blocks with opcode-like instructions, explicit physical stack operations
  (`dupN`, `swapN`, `pop`), and explicit terminators such as jumps, returns,
  reverts, and stops. Use it for target-specific CFG simplification, terminal
  block deduplication and tail merging, cold/revert-path handling, backend
  peepholes, computation and constant outlining, block layout, and
  address-sensitive code placement.
- Stack scheduling belongs in the MIR-to-EVM lowering boundary. Keep MIR value
  identities and virtual stack layouts in the scheduler's private representation,
  materialize `dupN`/`swapN`/`pop`, and emit already-scheduled EVM IR directly.
- Keep the assembler primitive. Lower block EVM IR once into a compact stream
  containing only opcodes, label definitions/references, deferred pushes, and
  immutable placeholders. The assembler resolves deferred values, computes the
  least fixed point of label offsets and PUSH widths, and emits bytes. PUSH
  widths cannot generally be selected in one forward pass because widening one
  forward reference can move a later target across another width boundary.
- Do not add CFG cleanup, peepholes, deduplication, outlining, layout, or other
  optimization logic to the compact assembly stream. Add those transforms to
  block EVM IR, where control-flow edges and block identity remain explicit.
- Keep the layers separate: MIR should not grow EVM stack-layout details, and
  EVM IR should not rediscover high-level Solidity typing or call semantics.

### MIR Phases

MIR is a phased IR, like rustc's MIR: a `Module` carries a `MirPhase`, phases
only move forward (the enum order is the lowering order), and the phase
round-trips through the text format as `@module Name` and `@phase ...` (printed
only when not the default). The phases, in order:

- `built`: fresh from HIR lowering — one MIR function per Solidity function,
  typed values, dispatch and ABI handling not yet materialized as MIR.
- `optimized`: the canonical pass pipeline has run
  (`run_default_pipeline_with_options` is the phase transition; ad-hoc
  `mir-opt` pass lists do not advance the phase).
- `abi`: each external function is a self-decoding wrapper — it decodes
  calldata into typed arguments and calls the original body as an internal
  function; the body keeps its fused external termination. Produced by the
  `lower-abi` pass.
- `dispatch`: the selector switch is an ordinary MIR `entry` function routing to
  the ABI wrappers through `tail_call` terminators (control transfers and does
  not return, matching the wrappers' external termination). Produced by the
  `lower-dispatch` pass, which requires the `abi` phase.
- `evm-shaped`: every call edge either returns or is an explicit `tail_call`
  (arguments included), the shape the backend expects. Produced by the
  `lower-evm-shaped` pass; argument-carrying tail calls are only formed for
  callees the backend statically frames, so their arguments store at
  compile-time frame addresses with no return address pushed.

The `lower-abi`, `lower-dispatch`, and `lower-evm-shaped` passes are progressive MIR-to-MIR lowering,
moving dispatch and ABI handling out of the backend. They run **by default** in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paradigmxyz/solar](https://github.com/paradigmxyz/solar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
