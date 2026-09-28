---
trigger: always_on
description: A C3-native JavaScript engine. **Goal**: pass 100% of the targeted test262 subset (the ~29,500 executable tests left after the skip list; roadmap in `plans/040-test262-100-percent.md`), beat Duktape on performance, keep memory low, and run on low-powered devices across platforms.
---

# Boomkat

## Project Spec

A C3-native JavaScript engine. **Goal**: pass 100% of the targeted test262 subset (the ~29,500 executable tests left after the skip list; roadmap in `plans/040-test262-100-percent.md`), beat Duktape on performance, keep memory low, and run on low-powered devices across platforms.

- Uses Duktape v2.7.0 and QuickJS as architectural references; leverage C3's native features for memory safety and its stdlib. When a path is unclear, compare Duktape source against QuickJS. Check the stdlib reference for what is available when planning a new feature.
- Focus on ES5/ES6 core; ignore *staging* features in the spec.
- RegExp uses libregexp (from QuickJS).
- **BigInt** is arbitrary precision: a 32-bit limb vector with `BIGINT_MAX_LIMBS = 1 << 26` (~2 billion bits, `src/hbigint.c3`). Magnitude is not a limit.
- **Strict vs sloppy**: scripts default to sloppy (matching ES2024); modules default to strict; per-function `is_strict` is recorded in `FuncFlags`. The `boomkat` CLI runs its input as a module unless given `--script`. The C API's `bk_eval` defaults to sloppy, and `bk_set_strict` switches a context to strict script code.
- **test262 skip list**: ~60% of test262 falls outside this engine's scope, which is ES5/ES6 core plus the sloppy-mode and Annex B behavior it needs (ECMA-402, Stage 3 proposals, host-specific and cross-realm behavior stay out). Scope is documented in `docs/engine-scope.md`. The skip list (`SKIP_DIRS`/`SKIP_GLOBS`/`SKIP_FILES`/`UNSUPPORTED_PATTERN`) is embedded directly in `scripts/run_test262.py`: update it there when implementing new features. The skip list is the *only* place scope is expressed — test selection itself is exhaustive over test262's directory tree, so a feature is out of scope because a rule names it, never because nobody listed its directory. `intl402` (ECMA-402) is skipped per test262's own guidance; `annexB` runs (920/1086 passing, 0 failures, 166 skipped: the Annex B String HTML wrappers, `legacy-regexp`, and `IsHTMLDDA` — see the sloppy-mode section); `staging` runs, as upstream `INTERPRETING.md` asks.

## Strict and Sloppy Modes

Sloppy-mode execution is a peer to strict mode. `plans/083-sloppy-mode.md` records how it was built.

**Current state:**
- `FuncFlags.is_strict` bit 7, plumbed through `CompilerContext.is_strict`. The legacy `Lexer.strict_mode` (octal rules) and `Lexer.reserved_words_strict` (keyword reservation) flags still exist. `subst_global_this` is gone — the predicate is `!is_strict() && !is_arrow()` at every call / construct / generator-create site.
- Top-level scripts default to sloppy (ES2024 §16.2.1.1); modules stay strict; ordinary functions and dynamic `Function` / `GeneratorFunction` / `AsyncFunction` bodies default to sloppy. `"use strict"` raises `is_strict`. Class code is strict throughout (ES2024 §11.2.2), so `class eval {}` is an early error even in a sloppy script.
- Sloppy-only syntax accepted: `with`, legacy octal literals and octal escapes, `delete <id>`, plain duplicate params (Annex B.3.1), duplicate `__proto__:` keys (Annex B.3.1), `for (var x = 1 in y)` (Annex B.3.5: a `var` ForBinding only, its initializer evaluated once before the RHS), labelled function declarations (Annex B.3.2), function declarations as `if` bodies (Annex B.3.4), and `let`/`static`/`yield` as identifiers.
- Still strict (unconditional): class / object method / arrow / named-export param duplicates (UniqueFormalParameters, no Annex B exemption), catch / lexical ForDeclaration duplicate BoundNames, `eval` / `arguments` as binding identifiers.
- Runtime semantics: implicit globals; `this` substitution plus primitive-to-wrapper boxing for a sloppy callee; DELPROP and DELVAR failing silently in sloppy (Annex B.3.1 result rules); the `arguments.callee` / `caller` poison pill; mapped `arguments` (Annex B.3.1); Annex B.3.3 for function declarations in a block, extended by B.3.2 (labelled) and B.3.4 (`if` body); a CallExpression assignment target (`f() = 1`, `f()++`, `for (f() of x)`) deferring to a runtime ReferenceError (Annex B.3.9); `with`-env semantics including `@@unscopables`, SnapshotReference stores, and dynamic name resolution in closures created inside the body.
- `annexB` passes 920 / 1086 with 0 failures and 166 skips. The legacy eval-code and global-code var-hoisting rules, the Annex B Date methods (`getYear`/`setYear`/`toGMTString`), and `catch (x) { for (var x …) }` redeclaration, are implemented. The 166 skips are scope exclusions, not gaps: the Annex B String HTML wrappers (111), `legacy-regexp` (26: `RegExp.$1`, `lastMatch`, `.compile()`), and `IsHTMLDDA` (29). The first two are legacy browser surface and are implementable — see plans/084; `IsHTMLDDA` (the `document.all` slot) is host-provided, so there is nothing to implement.

**What not to write:**
- Do not add new strict-only parse rejections.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ricardobeat/boomkat](https://github.com/ricardobeat/boomkat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
