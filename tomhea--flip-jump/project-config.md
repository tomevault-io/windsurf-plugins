---
trigger: always_on
description: This document is the authoritative convention list for the demonstration catalog
---

# Catalog Conventions

This document is the authoritative convention list for the demonstration catalog
under `programs/catalog/`. Read this end-to-end before writing or reviewing a
catalog program.

## Goal

A catalog of up-to-1200 distinct, small, working FlipJump programs that
collectively demonstrate the language is expressive enough to do anything.
Quality > quantity: a smaller catalog of meaningful programs beats a padded one.

## File layout

| Kind | Path |
|---|---|
| Source | `programs/catalog/<category>/<slug>.fj` |
| Input | `tests/inout/catalog/<category>/<slug>.in` |
| Expected output | `tests/inout/catalog/<category>/<slug>.out` |
| Compiled (generated) | `tests/compiled/catalog/<category>/<slug>.fjm` |

- **No `NNNN_` prefix in filenames.** Slugs are clean (e.g. `factorial.fj`).
- **Slugs are unique across the entire catalog**, not just within a category.
  This is forced by the pytest-name uniqueness rule below.
- **Categories** are listed in `README.md`.

## Catalog number `#NNNN`

Each program has a four-digit catalog number. The number appears in:

- The source header: `// Program Name (#NNNN)`.
- The `README.md` row.

Numbers run `0001`–`1029` contiguously, grouped by category in the order the
categories appear in `README.md`, so each category occupies a consecutive
block.

## Program header — every `.fj` starts with this

```
// Program Name (#NNNN)

// 1-line description that exactly matches the README.md description.
// 0-3 additional context lines, each beginning with `// `.

main

def main {
    stl.startup
    // ... body ...
    stl.loop
}

// Helper macros below, each prefixed with a 1-line `// what it does` comment.
```

The `README.md` `description` column is the **single source of truth**; the
very first description line of each `.fj` must match it byte-for-byte.
Subsequent header lines may add context.

## `main` macro convention

- The macro is literally named `main` (not `<slug>_main`).
- The top-level non-macro code is essentially just the line `main`. The only
  other things allowed at top level are `segment` / `reserve` directives for
  programs that need explicit memory layout (per `programs/prime_sieve.fj`).
- Helper macros and variable declarations live below `def main { ... }`.
- **`stl.startup` and `stl.loop` appear ONLY inside `main`, never inside a
  helper macro.** They are the program-lifecycle bookends — the startup brings
  up the runtime, the loop halts it. A helper macro that contains either of
  them is misshapen: it's trying to be a program, not a function. (Look at
  the STL itself for the same rule — no `stl.macro` definition uses
  `stl.startup` or `stl.loop` in its body.)

## Sub-macros — "functionalize" the body

Programs more complex than a one-line `stl.output` should be decomposed into
helper `def`s, not crammed into `main`. `main` then reads as a sequence of
high-level calls. This isn't aesthetic — it's how the language is meant to be
used (every STL feature you call is itself a macro), and it pays back fast on
the medium-complexity programs (parsing, sorting, n-queens, etc.).

Practical rules of thumb:

- If `main`'s body is more than ~15 lines or has more than one `@`-label loop,
  pull each loop out into its own helper.
- A helper has a single clear job. Its name describes that job. Its `def` is
  preceded by a 1-line `// what it does` comment.
- Helpers receive their inputs via macro parameters and reference their
  external data via the `< ...` clause. They do NOT define their own
  `stl.startup` / `stl.loop` (see above).
- Loops over input bytes, decimal-digit parsing, case toggling, counter
  printing, etc. are all natural helper candidates and recur enough across
  the catalog that defining them once per program (and letting future programs
  copy the pattern) is the right call.

### Worked example

Don't:
```
def main @ loop, end < ch, counter {
    stl.startup
  loop:
    bit.input ch
    bit.if0 8, ch, end
    bit.inc 16, counter
    ;loop
  end:
    bit.print_dec_uint 16, counter
    stl.output '\n'
    stl.loop
}
```

Do:
```
def main < ch, counter {
    stl.startup
    count_input_bytes ch, counter
    bit.print_dec_uint 16, counter
    stl.output '\n'
    stl.loop
}

// Read bytes from stdin until \0, incrementing counter for each.
def count_input_bytes ch, counter @ loop, end {
  loop:
    bit.input ch
    bit.if0 8, ch, end
    bit.inc 16, counter
    ;loop
  end:
}
```

The "Do" version reads top-down as a recipe and the helper is reusable in
adjacent programs.

## Code style (CR-ist-enforced)

- Clean, easy-to-read macro and label names.
- Whitespace between logical sections.
- Helper macros under their `def` line have a `// what it does` comment, 1-2 lines.
- No dead code, no commented-out lines, no `TODO`s.
- Prefer `hex.*` macros over `bit.*` (4x smaller/faster).
- **For long repeats, use a runtime loop** (counter + cmp + branch).
  `rep` is the same as manual unroll; if compile time grows past ~5s, switch
  to a runtime loop.
- **NEVER name a label `n`, `i`, `w`, `dw`, or `dbit`.** These collide with STL
  macro internals: `n` is the width parameter in every `bit.*`/`hex.*` macro,
  `i` is the conventional `rep(n, i)` loop variable, and `w`/`dw`/`dbit` are

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomhea/flip-jump](https://github.com/tomhea/flip-jump) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
