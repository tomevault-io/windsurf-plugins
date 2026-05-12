---
trigger: always_on
description: This document provides guidance for AI agents working on the EvmAsm project.
---

# AI Agent Guide for EvmAsm

This document provides guidance for AI agents working on the EvmAsm project.

## Project Overview

EvmAsm is a verified macro assembler for RISC-V in Lean 4, inspired by "Coq: The world's best macro assembler?" (Kennedy et al., PPDP 2013). The project demonstrates using Lean 4 as both a macro language and verification framework for assembly code.

## Build System

- **Build tool**: Lake (Lean 4's build system)
- **Toolchain**: Lean 4.28.0-nightly-2026-01-22 (specified in `lean-toolchain`)
- **Build command**: `lake build`
- **Clean build**: `lake clean && lake build`

### Important Lake Configuration Notes

- The `lakefile.toml` uses Lake 5.0 format (root-level package fields, no `[package]` section)
- `defaultTargets = ["EvmAsm"]` is required for `lake build` to work
- The library name is `EvmAsm` and sources are in `EvmAsm/` directory

## Project Structure

```
EvmAsm/
  Rv64/                -- RV64IM machine model + infrastructure
    Basic.lean         -- Machine state: registers, memory, PC
    Instructions.lean  -- RV64IM instruction semantics (incl. ECALL)
    Program.lean       -- Programs as instruction lists, sequential composition
    Execution.lean     -- Step execution, code memory, ECALL dispatch
    SepLogic.lean      -- Separation logic assertions and combinators
    CPSSpec.lean       -- CPS-style Hoare triples, branch specs, structural rules
    GenericSpecs.lean  -- Generic instruction spec templates
    InstructionSpecs.lean -- Concrete instruction specs
    SyscallSpecs.lean  -- Spec database (@[spec_gen_rv64])
    ControlFlow.lean   -- if_eq macro, symbolic proofs
    ByteOps.lean       -- Byte-level: extractByte algebra, LBU/SB specs
    Tactics/           -- Automation: xperm, runBlock, liftSpec, etc.
  Evm64/               -- 256-bit EVM opcodes on RV64IM (4×64-bit limbs)
    Basic.lean         -- EvmWord (BitVec 256), getLimb/fromLimbs
    Stack.lean         -- evmWordIs, evmStackIs assertions
    Add/, Sub/, ...    -- Individual opcode implementations (30+ files)
  EL/                  -- Pure Ethereum Execution Layer specs
    RLP/               -- RLP encoding/decoding (no RISC-V dependency)
      Basic.lean       -- RLPItem type, encode
      Decode.lean      -- decode with canonical enforcement
      Properties.lean  -- Round-trip proofs (native_decide)
EvmAsm.lean            -- Root module: imports Rv64, Evm64, EL
```

## Key Lean 4 API Compatibility Notes

When working with this codebase, be aware of these Lean 4 nightly API changes:

1. **Logic lemmas**: Use lowercase names (`and_assoc`, `and_comm` instead of `And.assoc`, `And.comm`)
2. **Doc comments**: Cannot place `/-- ... -/` doc comments immediately before `#eval` commands (use regular `--` comments)
3. **Proof tactics**: `simp` may need explicit lemma lists or `rw` for manual rewriting
4. **Namespace**: Most theorems are in `namespace MachineState`, so use full names like `MachineState.getReg_setPC`

## Verification Workflow

When adding or modifying proofs:

1. **Build first**: Always run `lake build` to see current errors
2. **Use MCP tools**: The lean-lsp MCP server provides:
   - `lean_goal`: Check proof state at a position
   - `lean_diagnostic_messages`: Get compiler errors
   - `lean_hover_info`: Get type information
   - `lean_completions`: Get IDE completions
   - `lean_local_search`: Search for declarations locally
   - `lean_leansearch`: Natural language search in mathlib
   - `lean_loogle`: Type-based search in mathlib
3. **Test concretely**: Verify specific cases with `native_decide` before generalizing
4. **Incremental development**: Prove helper lemmas before the main theorem

## Critical Rules

- **Naming convention (Mathlib-aligned):**
  - **camelCase** for *value* identifiers: `let`-bound locals, theorem/lemma parameters, function arguments, definitions (e.g. `let qAddr := ...`, `theorem foo (carryNat : Nat)`).
  - **snake_case** for *hypothesis* names — proof bindings introduced by `have h_… : Prop`, `obtain ⟨h_lt, h_eq⟩`, `intro h_pos`, etc. Mathlib keeps these snake_case (e.g. `h_pos`, `h_le`, `h_zero`, `h_eq`). Do **NOT** rename `h_*`-style hypothesis names to camelCase as part of #189 cleanup — that's the wrong direction. PR #1497 made this mistake.
  - When in doubt: if it names a `Prop`-typed term used in a proof, leave snake_case; if it names data (a `Nat`, `Word`, `BitVec`, etc.), use camelCase.

- **Do NOT add `set_option maxHeartbeats` to any file** unless you are in `Evm64/Shift/` composition files (Compose, ShlCompose, SarCompose) for body/path composition proofs. Heartbeat limits are configured globally in `lakefile.toml`.
- **Do NOT add `set_option maxRecDepth` to any file.** Recursion depth is configured globally in `lakefile.toml`.
- If a proof times out or hits recursion limits, restructure the proof (e.g., split into smaller lemmas, use intermediate `have` bindings) rather than increasing limits. Increasing `maxRecDepth`/`maxHeartbeats` is almost always a waste of time — the real issue is typically a unification mismatch, wrong argument order, or missing address canonicalization.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Verified-zkEVM/evm-asm](https://github.com/Verified-zkEVM/evm-asm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
