---
trigger: always_on
description: `llvmkit` is a from-scratch Rust reimplementation of LLVM IR APIs. It is **not** an FFI binding to `libLLVM` — the build and runtime never depend on `libLLVM` or `llvm-sys`.
---

# Repository Guidelines

## Project Overview

`llvmkit` is a from-scratch Rust reimplementation of LLVM IR APIs. It is **not** an FFI binding to `libLLVM` — the build and runtime never depend on `libLLVM` or `llvm-sys`.

Goals, in priority order:

1. **Read and write LLVM IR** (textual `.ll` first, bitcode later) with idiomatic Rust I/O traits.
2. **Provide an `IRBuilder` analog** for programmatic IR construction.
3. **Mirror LLVM's logic exactly**, using the C++ source under `orig_cpp/` as the canonical reference for behavior.
4. **Make invalid IR unrepresentable** at the type level wherever LLVM uses runtime checks. Where C++ forces `if (v->getType()->isFloatTy())`, Rust should expose a sum type whose variants already encode the answer.

What `llvmkit` is *not*:

- Not a binding crate (`llvm-sys`, `inkwell`, `llvm-ir`-style wrappers are all out of scope).
- Not a code generator and not a target backend. `llvmkit` doesn't lower IR to machine code or link objects — use upstream LLVM (`llvm-sys`, `inkwell`) for that. Optimization / transform / analysis passes are *planned* future work, not excluded; they will land once the IR data model, builder, parser, and verifier are stable.

## Project Status

The repo is a Cargo workspace at `C:/Users/Aslan/llvmkit/`. Implemented today:

- The `.ll` lexer (`llvmkit-asmparser/src/ll_lexer.rs`).
- The IR data model with **width-typed integers** (`IntType<'ctx, W>`, `W in { bool, i8, i16, i32, i64, i128, IntDyn, Width<const N: u32> }`) and **kind-typed floats** (`FloatType<'ctx, K>`, `K in { f32, f64, Half, BFloat, Fp128, X86Fp80, PpcFp128, FloatDyn }`).
- Sealed marker traits: `IntWidth`, `StaticIntWidth`, `FloatKind`, `StaticFloatKind`, `WiderThan`, `FloatWiderThan`, `ReturnMarker`, `SelectArm`.
- Multi-source operand traits: `IntoIntValue<W>`, `IntoFloatValue<K>`, `IntoPointerValue`, `IntoConstantInt<W>`, `IntoConstantFloat<K>`, `IntoReturnValue<R>`.
- The full medium IRBuilder: every integer binop (`add`/`sub`/`mul`/`udiv`/`sdiv`/`urem`/`srem`/`shl`/`lshr`/`ashr`/`and`/`or`/`xor`) plus per-opcode flag types (`AddFlags`/`UDivFlags`/...), every float binop (`fadd`/`fsub`/`fmul`/`fdiv`/`frem`), every cast (`trunc`/`zext`/`sext`/`bitcast`/`ptrtoint`/`inttoptr`/`fptrunc`/`fpext`/`fptosi`/`fptoui`/`sitofp`/`uitofp`/`addrspacecast`), `icmp`/`fcmp`, control flow (`br`/`cond_br`/`unreachable`), `phi` (chainable `add_incoming`), memory (`alloca`/`load`/`store` with optional `Align`), `getelementptr` (`build_gep`/`build_inbounds_gep`/`build_struct_gep`), `call` (flat + chainable `CallBuilder`), and `select` (sealed `SelectArm` for int/float/pointer arms).
- AsmWriter producing real `.ll` output via `format!("{module}")` for every shipped opcode.
- **Verifier** (`crates/llvmkit-ir/src/verifier.rs`): `Module::verify_borrowed(&self) -> IrResult<()>` for diagnostic-only validation and `Module::verify(self) -> IrResult<VerifiedModule<'ctx>>` for the typestate path that brands a module as well-formed. Implements the constructive subset of `llvm/lib/IR/Verifier.cpp::visit*` for every shipped opcode (binary int/float, icmp/fcmp, all casts, alloca/load/store, GEP, call, select, phi, ret/br/unreachable). Catches type mismatches, terminator placement, phi-predecessor coherence, ambiguous phi, in-block use-before-def, and self-reference. Cross-block dominance is deferred to Session 4 (DominatorTree).
- **Mutation API (T1)**: full instruction-lifecycle typestate. `Instruction<'ctx, S = state::Attached>` is parameterised by `S: state::InstructionState` (sealed; variants `Attached` / `Detached`). `Instruction` is intentionally **`!Copy` and `!Clone`** (Doctrine D2): the linear-typed handle prevents use-after-erase / double-erase at compile time. Per-opcode handles (`AddInst`, `LoadInst`, ...) stay `Copy` and now hold `(ValueId, ModuleRef, TypeId)` directly; their `as_instruction(self)` materialises a fresh `Instruction<Attached>` on demand. Every operand slot in `instr_types.rs` is wrapped in `core::cell::Cell<ValueId>` (and `Cell<Option<ValueId>>` for the optional `alloca` / `ret` operands) so RAUW can rewrite the operand wiring through `&self`. The shipped lifecycle methods are:
    - `Instruction<Attached>::replace_all_uses_with(self, replacement)` --- `Value::replaceAllUsesWith` in `lib/IR/Value.cpp`.
    - `Instruction<Attached>::erase_from_parent(self)` --- `Instruction::eraseFromParent`.
    - `Instruction<Attached>::detach_from_parent(self) -> Instruction<Detached>` --- `Instruction::removeFromParent`.
    - `Instruction<Attached>::move_before(self, other)` / `move_after` --- `Instruction::moveBefore` / `moveAfter`.
    - `Instruction<Detached>::insert_before(self, other)` / `insert_after` / `append_to(block)` --- `Instruction::insertBefore` / `insertAfter` / `insertInto`.
    - `Instruction<Detached>::drop_detached(self)` --- discard a detached instruction without inserting it (deregisters its operands' use-list entries).
    - `BasicBlock::splice_into(self, dest)` --- `BasicBlock::splice`.
    - `BasicBlock::split_at(self, before, name) -> BasicBlock<R>` --- `BasicBlock::splitBasicBlock`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [r3bb1t/llvmkit](https://github.com/r3bb1t/llvmkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
