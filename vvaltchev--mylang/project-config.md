---
trigger: always_on
description: handles ride inside the moved Instr structs). Iterated <=4 rounds, each:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

> **READ `README.md` IN FULL BEFORE TOUCHING ANYTHING.** This is a small project
> and the README is
> the complete language specification (every keyword, every builtin, every
> semantic rule, with
> examples). It is not optional reference material to consult on demand — read
> the whole thing up
> front, once, so you know the language the interpreter implements. This
> CLAUDE.md covers the *C++
> implementation*; the README covers the *language*. You need both in your head
> before making changes.

> **KEEP THE DOCS IN SYNC WITH THE SOURCE — IN THE SAME CHANGE.** `README.md`
> and this `CLAUDE.md`
> are part of the codebase, not afterthoughts. Any code change must carry its
> documentation update in
> the *same commit*, never as a follow-up:
> - **`README.md`** whenever script-visible behavior changes — a
>   new/removed/renamed keyword, builtin,
>   operator, or numeric constant; changed semantics; new error conditions.
>   README is the language
>   spec; if it and the interpreter disagree, that's a bug.
> - **`CLAUDE.md`** whenever the *implementation* shape changes — a new source
>   file or `.cpp.h`, a new
>   `TypeE`/AST node, a changed design rule or invariant, a new convention, or
>   anything that would make
>   a sentence in this file wrong. After editing code, reread the relevant
>   CLAUDE.md section and fix any
>   statement the change just falsified.
>
> A change that alters behavior or architecture but leaves the docs stale is
> incomplete.

## What this is

MyLang is an educational, dynamically-typed scripting language (C-looking
syntax, Python-ish
semantics) implemented as a tree-walking interpreter in portable C++17. It has
**no dependencies**
beyond the standard library, including for its tests. The single `mylang`
executable both compiles
(lex + parse + const-fold) and runs scripts. Author's goal was to have fun
writing a recursive-descent
parser; correctness and clarity matter more than raw speed, though performance
shaped several core
design choices (see the value model below).

## Build & run

```
make -j                    # release build (-O3) -> build/mylang
make -j TESTS=1 OPT=0      # debug build, unit tests compiled in (for -rt)
make -j BUILD_DIR=other    # out-of-tree build
make clean
```

`OPT` defaults to 1 (`-O3`); `OPT=0` drops it. `TESTS=1` adds `-DTESTS`, which
is what compiles the
`-rt` suite into the binary. Base flags:
`-std=c++17 -ggdb -Wall -Wextra -Wno-unused-parameter
-fwrapv`. The Makefile auto-generates header dependencies under `.d/`.

**LTO is on by default for optimized builds.** `LTO` defaults to `OPT`, so a
release build links with `-flto=auto` (added to `BASE_FLAGS`, which the link
line passes too) — ~7% smaller binary and ~8-9% faster on `bench/`. It works on
both GCC and clang and is verified to keep `-rt` green. Build with `LTO=0` to
disable (e.g. for a faster/debuggable link); an `OPT=0` build is non-LTO anyway.

**Sanitizers default on for debug builds.** `ASAN` and `UBSAN` (AddressSanitizer
/ UndefinedBehaviorSanitizer) both default to **on when `OPT=0`** and **off when
`OPT=1`**, and either can be forced: `make ASAN=0` (debug, no ASan),
`make OPT=1 UBSAN=1` (sanitized release). The flags go into `BASE_FLAGS` so they
reach the compile and link lines. UBSan is configured `-fno-sanitize=signed-
integer-overflow`, because the codebase relies on `-fwrapv` wraparound (that
overflow is *defined* here, not a bug); it also runs with
`-fno-sanitize-recover=undefined`, so a UBSan finding **aborts** (non-zero exit)
instead of diagnose-and-continue — otherwise a real UB could print and still
exit 0 past CI's exit-code check. `-fno-omit-frame-pointer` is added whenever
either sanitizer is on. `-rt` is verified green under both.

**Assertions: `ASSERTS` (default 1).** The C `assert()` + the project's
`ML_CHECK()` invariant net (see *Invariants & hazards*) are **on for every build
type** (debug AND release), so every build and CI lane exercises them. With
`ASSERTS` on the build also enables libstdc++ container hardening
(`-D_GLIBCXX_ASSERTIONS`, ABI-safe; the libc++ analog is set per-OS in CI).
`make ASSERTS=0` defines `-DNDEBUG`, compiling all of that away — use it on an
optimized build to measure the assertion overhead, e.g. `make OPT=1 ASSERTS=0`
vs the default `make OPT=1`. **`RECYCLE` (default 0):** `make RECYCLE=1 TESTS=1`
builds the adversarial node allocator (see *Invariants & hazards*).

**Warnings-as-errors: `WERROR` (default 1).** Every build (both build systems,
every type, ALL THREE compilers) treats a warning as an ERROR, so a warning
CANNOT be pushed — it fails the build and must be addressed (see the warning
rule under *Conventions*). GCC/Clang get `-Werror`; MSVC gets `/WX` (its CMake
default level is `/W3`). CMake also gains the Makefile's `-Wall -Wextra
-Wno-unused-parameter` for GCC/Clang (it set none before, so CI was laxer than a
local build — that gap is why the MSVC narrowing/`getenv` warnings accumulated
unnoticed). `make WERROR=0` (or CMake `-DWERROR=OFF`) opts out for a
work-in-progress build.

**VM hardening: `VM_HARDENING` (default = debug).** The heavy per-op VM

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vvaltchev/mylang](https://github.com/vvaltchev/mylang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
