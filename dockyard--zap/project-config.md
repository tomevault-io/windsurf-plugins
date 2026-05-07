---
trigger: always_on
description: **STRICTLY FORBIDDEN.** Never implement workarounds, hacks, temporary fixes, or shortcuts of any kind. Every solution must be the correct, production-grade, long-term fix — regardless of how difficult, expensive, or time-consuming it is. If a proper fix requires deep architectural changes across multiple files, that is the fix. If it requires changes to the Zig fork, make those changes. If it requires redesigning a data structure, redesign it. Cost and time are not concerns — correctness and qua
---

## No Workarounds, Hacks, or Shortcuts

**STRICTLY FORBIDDEN.** Never implement workarounds, hacks, temporary fixes, or shortcuts of any kind. Every solution must be the correct, production-grade, long-term fix — regardless of how difficult, expensive, or time-consuming it is. If a proper fix requires deep architectural changes across multiple files, that is the fix. If it requires changes to the Zig fork, make those changes. If it requires redesigning a data structure, redesign it. Cost and time are not concerns — correctness and quality are. Never choose expediency over correctness. Never paper over a root cause with a band-aid. If you find yourself writing a "temporary" fix, stop — find and implement the real solution.

## Never compromise on code quality

When you have an option to finish a taks faster but implementing a short cut, a hack, or some lower quality solution nerver take this option. Always pursure the highest quality solution regardless of the implementation cost or complexity of the solution.

## Working with our Zig fork

Zap uses a fork of Zig 0.16.0 which should be at ~/projects/zig

You have full permission to make changes to this fork if necessary to unblock work in Zap. The same code quality rules apply to our Zig fork as does this project.

When you need to compile our Zig fork consider the compilation guide in Zap's README as well as the README in our Zig for to understand how to build our Zig fork to compile Zap.

## Zap is a Language — Implement Features in Zap Code

**THIS IS THE MOST IMPORTANT RULE. VIOLATIONS ARE UNACCEPTABLE.**

Zap is a general-purpose programming language. Features, behaviors, library functions, and language constructs MUST be implemented in Zap source code (`lib/*.zap`), NOT hardcoded in the Zig compiler (`src/*.zig`).

**NEVER hardcode Zap struct names, function names, or library behavior in the compiler.** The compiler is a general-purpose tool. It does not know about IO, String, Kernel, Map, Zest, or any other Zap struct. If you find yourself writing a Zap struct name as a string literal in Zig source, you are doing it wrong. Stop. Think. Find the Zap-level solution.

**ALWAYS attempt the Zap solution first.** Before touching any Zig compiler code, ask: "Can this be done in Zap?" If the answer is "yes" or "maybe," do it in Zap. If you think it can't be done in Zap, think harder. Research how Elixir solves it. Research how other languages solve it. Only touch the compiler as a last resort for genuine language primitives (parsing, type system, ZIR emission).

**The only things that belong in Zig:**
- Lexer/parser syntax (tokens, AST nodes)
- Type system primitives (Bool, String, Atom, i64, etc.)
- ZIR emission mechanics
- Zig runtime primitives that physically cannot be expressed in Zap (stdout, OS argv, memory allocation)

**Everything else is Zap code:**
- Standard library functions (IO, String, Integer, etc.) — defined in `lib/*.zap`, call `:zig.` for primitives
- Macros (if, unless, and, or, |>, sigils) — defined in `lib/kernel.zap`
- Test framework — defined in `lib/zest/*.zap`
- Sigil implementations — defined as macros in Kernel
- Validation, error checking, behavior — Zap macros and functions

**Do not take shortcuts.** Do not bypass Zap because the Zig solution is "easier" or "faster." Do not hardcode behavior in the compiler because you can't figure out how to do it in Zap. Spend the time. Research deeply. Find the correct solution. Cost and time are not concerns — correctness is.

**Do not create unnecessary abstractions in Zig.** No `@native`, no route tables, no runtime state tracking in Zig for things that are Zap-level concerns. If something was implemented in Zig and it could be Zap, rip it out and rewrite it in Zap.

## Documentation

**All public Zap functions MUST have `@fndoc` attributes.** Every `pub fn` and `pub macro` in `lib/*.zap` files must have a `@fndoc` string describing what it does. Use heredoc `"""` for multi-line docs. No exceptions.

## Zap Code Quality

- **Blank line after every heredoc closing `"""`**. The `"""` must be followed by an empty line before the next declaration or attribute. No exceptions.
- **`@structdoc` goes inside the struct body**, immediately after `pub struct Name {`.
- **`@fndoc` goes immediately before the function/macro it documents**, with a blank line between the closing `"""` and the `pub fn`/`pub macro`.
- **All `@fndoc` and `@structdoc` use heredoc `"""`** for multi-line content.
- **Escape `#{` in doc examples** as `\#{` to prevent interpolation inside heredocs.
- **Always use descriptive names.** Never use short or cryptic variable names, parameter names, or helper names when writing new code. Prefer explicit names that make the code readable without extra context.

## Development Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DockYard/zap](https://github.com/DockYard/zap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
