---
trigger: always_on
description: **2026-07-31:** This is the condensed operating manual (~300 lines). The full
---

# Briev Compiler — Agent Guidelines

**2026-07-31:** This is the condensed operating manual (~300 lines). The full
pre-rewrite document is preserved in `AGENTS.md.archive`; reference material
(language syntax, contracts, coding standards, backend architecture) lives in
`docs/architecture/agent-reference.md`. Historical context: `AGENTS_HISTORY.md`,
`AGENTS_HISTORY_2.md`.

## Operating Contract

You are building a compiler that must be correct for **all programs** written
in Briev, not just the test case in front of you. Zero tolerance: "probably
fine" is a critical failure. Every edge case, undefined behavior, or bug in a
file you touch is solved completely NOW — never deferred, never "out of scope,"
never "pre-existing."

Every decision passes three questions:

1. **Does this make the compiler more general, or special-case one pattern?**
   A match arm for `"ring_push"` solves today's benchmark; tomorrow's
   `MyQueue<T>` with `InsertAt <~ my_push(#Lh, #Rh)` demands the same treatment.
2. **Does this add knowledge the compiler must carry forever, or push it into
   configuration/stdlib where it can evolve?** The dividing line is
   `--no-stdlib`: if it must work without stdlib, it's an intrinsic; everything
   else belongs in config or `.bv` files.
3. **If this were the only rule left, would the architecture still hold?**
   Removing any one rule must not break the others.

Patches are unacceptable. There is no "go fast and break things."

## Golden Rules

1. **CONTRACT-FIRST**: Contracts are the source of truth. Never weaken
   `[product > 0]` to `[true]` — fix the code, not the contract.
2. **MAXIMUM EFFICIENT DEFAULT**: The compiler MUST pick the most efficient
   codegen strategy for EVERY program automatically — every case, not just the
   benchmark at hand. This covers every codegen decision: tuple slot
   allocation, collection strategy, probe cost, materialization, loop shape.
   A user should never need a strategy keyword to reach competitive
   performance; a keyword-beaten default is a compiler bug (fix the default,
   never let the modifier carry the win). Strategy keywords (`seq`, `vol`,
   `pack`, `async`, `sync<g>`, `atomic`, `union`, `trap`) exist for a
   DIFFERENT purpose — to express intended behaviour that plain efficient
   codegen cannot: `seq` forces a precise declaration order or sequential
   execution that aggressive vectorization/parallelism would break, `vol`
   demands volatile memory the optimizer may not eliminate, others express
   required embedded/inter-language semantics. They are for correctness and
   intent, never for speed. Requiring a keyword to win is a failing default.
3. **NO OBFUSCATION OF SPECIAL TREATMENT**: The compiler has intrinsics,
   hashwords, reflection, and directives — they exist, and pretending they
   don't is a purist trap. What is forbidden is HIDING them behind
   ordinary-looking syntax. Two-part principle:
   - **Avoid accidental complexity.** Essential complexity (SMT, LLVM IR
     emission) is unavoidable and kept. Accidental complexity (heuristic
     trees, hand-rolled passes that fight the design) is stripped, never
     preserved.
   - **Disclose special treatment.** Every compiler-known behavior carries an
     explicit marker: `#` suffix (intrinsic `Sqrt#`), `#` prefix (hashword
     `#Int`), `!` suffix (compile-time expansion `my_macro!`), `.^`/`.^^`
     (reflection). User-facing directives (`seq`, `pack`, `vol`, `async`,
     `sync<g>`, `atomic`, `union`, `trap`) are ordinary keywords — no `#` —
     disclosed at use (their purpose is correctness/intent, never speed —
     see Rule 2).
   - **NEVER hardcode Rust string matches as built-in functions.**
     `is_digit` → `import char from "std/char.bv"`. Primitive types (Int,
     Float, Bool, Ptr, Void) are the sole bootstrap exceptions.
4. **INTRINSICS BEFORE FRGN**: Check `get_intrinsic_signature()` before writing
   `frgn`. All intrinsic names are PascalCase + `#` suffix (`Sqrt#`).
5. **INTERPRETER IS REFERENCE**: If the interpreter runs it correctly, the
   backend must compile it. Fix codegen, never the interpreter.
6. **ADDITIVE ONLY**: Never modify existing optimization paths — new match arms
   only. The `_ => return None;` fallthrough must remain unchanged.
7. **ALWAYS FINISH**: No `todo!()`, `unreachable!()`, `// TODO:`, or stubs in
   committed code. Every feature wired parser → AST → analysis → codegen → tests.
8. **NEVER DISCARD UNCOMMITTED WORK**: `git checkout --`, `git restore`, and
   `git checkout .` DESTROY work permanently — never use them. Commit your own
   changes with targeted `git add`; never stash others' work. `git reset HEAD`
   is safe (unstaging only).
9. **TESTS OR IT DOESN'T EXIST**: Every feature, code path, and match arm needs
   tests. `cargo test --lib` before every commit.
10. **NO PROTOTYPING**: Every optimization is a first-class pass in its proper
    module — never inline analysis into codegen as a shortcut.
11. **EXECUTIVE REQUESTS ARE NOT OPTIONAL**: Told to fix a pattern? Do all of
    it. If prereqs are missing, implement them first.
12. **PLAN WITH BENCHMARKS**: Every performance plan MUST include a baseline
    table of ALL benchmark results at the current commit BEFORE changes, and the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Randozart/briev-lang](https://github.com/Randozart/briev-lang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
