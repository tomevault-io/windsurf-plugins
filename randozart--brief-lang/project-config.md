---
trigger: always_on
description: See CLAUDE.md for complete documentation. This file ensures OpenCode picks up the same guidelines.
---

# Brief Compiler - Agent Guidelines

See CLAUDE.md for complete documentation. This file ensures OpenCode picks up the same guidelines.

## Quick Reference

### Commands
- **Build**: `cargo build`
- **Test**: `cargo test --lib`
- **Test backend registry**: `cargo test --lib -- backend::tests`
- **Compile RBV**: `./target/release/brief-compiler rbv <file.rbv>`
- **Benchmark**: `bash benchmarks/build_and_bench.sh` — always use this harness (nanosecond CLOCK_MONOTONIC, 5-iteration average). Ad-hoc timing produces false hangs and imprecise numbers.

### File Types
- **.bv** - Brief (standard Brief file, cosmopolitan tier — any FFI, any language, OS assumed)
- **.sbv** - Strict Brief (full contracts required, no sugar defaults)
- **.rbv** - Rendered Brief (Brief + View, compiles to web frontend. Like `.tsx` is to `.ts`)
- **.srbv** - Strict Rendered Brief (full contracts required in web target)
- **.ebv** - Embedded Brief (bare metal — no OS, no GC. C/Rust FFI allowed but Python/Java warned)
- **.sebv** - Strict Embedded Brief (full contracts required, bare metal)
- **.hebv** - Hardware Embedded Brief (pure logic graph — no FFI, no external deps, only synthesizable types. Contracts must be total. Outputs Verilog/VHDL/SV)
- **.dbv/.dbvs/.dbvl** - Data Brief (configuration with schema, think `.xml`/`.xmls`/`.jsonl`)

### Contract Sugar Syntax

Brief provides sugar for single-sided contracts. Use these where possible in the stdlib
to teach readers the pattern:

| Syntax | Precondition | Postcondition | Meaning |
|---|---|---|---|
| `[pre][post]` | `pre` | `post` | Full contract (both sides) |
| `[[post]` | `true` (omitted) | `post` | Postcondition only, no guard. **The opening `[[` means the precondition was omitted.** |
| `[pre]]` | `pre` | `true` (omitted) | Guard only, no guarantee. **The closing `]]` means the postcondition was omitted.** |

Memory aid: the left bracket `[` is always the precondition. `[[` = two left brackets = the
first one opens an empty precondition (defaults to `true`), the second opens the postcondition.
`]]` = two right brackets = the first closes the precondition, the second closes an empty
postcondition (defaults to `true`).

**Banned in**: `.sbv`, `.srbv`, `.sebv`, `.hebv` (strict tiers require explicit both-sided contracts).

**Preferred in**: `.bv`, `.ebv`, `.rbv` stdlib examples. Use the sugar to keep code readable
while teaching users the pattern.

### Critical Philosophy

**CONTRACT-FIRST**: Contracts are the source of truth. Never weaken contracts to match lazy code.

**NO MAGIC**: Never add hardcoded Rust string matches as "built-in" functions.
- If a `.bv` file needs `is_digit`, import `char` from `"std/char.bv"` — NOT a Rust match arm.
- If a `.bv` file needs `None`, import `option` from `"std/option.bv"` — NOT pre-populating state.
- The FFI system (`frgn from "..."`) and the standard library are the transparent paths. Use them.

**SELF-DOCUMENTING FAILURE**: Before fixing any issue:
1. Understand WHY the fix works (not just THAT it works)
2. Document the root cause in BUGS.md
3. Ensure the fix doesn't violate Contract-First or No Magic

### Anti-Patterns (NEVER DO)
- Changing `[product > 0]` to `[true]` because code doesn't set product
- Using generic contracts like `[true]` that pass everything
- Adding postconditions that don't guarantee specific outcomes
- Adding Rust string-match built-ins when the standard library or import system should be used
- Pre-populating interpreter state with enum constants (None, Some, Ok, Err)
- Adding `x == x` self-references in preconditions to force liveness
- Adding synthetic exit-condition fields solely to prevent dead-field elimination
- **Hardcoded `from` strings**: `from "libruntime"` is magic — parsed and discarded. Use `from "c"` or omit `from` entirely (symbol resolves from `import "link/..."` targets).
- **Hardcoded runtime declares**: `__rt_init`, `__rt_wait`, etc. must be declared as `frgn` in `std/rt.bv` and imported by the user — never hardcoded in `emit_declares()`.
- **Name-based interpreter dispatch**: Matching on `fn_name == "insert"` instead of dispatching on `Value::HashMap` — dispatch on the type, not on a string.
- **`"None"`/`"Err"` discriminant magic**: Never match on variant names for discriminants. Use the enum declaration order.
- **Type-based dispatch**: In the interpreter, dispatch on `Value` variant, not on string-matching the function name.

### Observability as Liveness

A program that produces no observable effect IS dead code. The compiler is correct to eliminate it.

Brief's liveness model: **a value is live if an FFI call consumes it.** Every program must interact with the world — print, file I/O, network — via `frgn` calls.

If the compiler folded your hot loop to `store i64 N`, **the compiler is right.** Your program produced no observable output. The fix is NOT liveness hacks (`x == x`, synthetic exit fields). The fix IS `frgn __print_int(result)`.

The C reference must use the SAME observable. Symmetric benchmarks, symmetric optimizations.

#### `term! -> swan_song` is the correct liveness pattern for terminal programs

When a program must run a specific FFI call (print, write) as its final act before exiting,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Randozart/brief-lang](https://github.com/Randozart/brief-lang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
