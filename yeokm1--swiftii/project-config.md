---
trigger: always_on
description: This file orients AI coding assistants (primarily **Claude Code** and
---

# AGENTS.md - Working with SwiftII

This file orients AI coding assistants (primarily **Claude Code** and
**Codex**, which did much of the work on SwiftII, plus Cursor and
others) and human collaborators to the SwiftII project. **Read this and
[`CONSTRAINTS.md`](docs/contributing/CONSTRAINTS.md) at the start of every session
before writing or editing code.** They are short on purpose.

---

## What we're building

SwiftII is a Swift-flavored language interpreter for the Apple II. The
binding target is the Apple II Plus with a 16K language card (64K
total) - the hardest fit and the one budgets are set against. The //e
(aux RAM, 80-column) and Saturn 128K cards get more headroom and extra
features, and the original ][ boots too; see
[`CONSTRAINTS.md`](docs/contributing/CONSTRAINTS.md) for the per-machine picture.

It supports three execution paths:

1. **REPL** - user types Swift expressions and statements at a prompt,
   they execute immediately, state persists across the session.
2. **File** - the interpreter reads a `.swift` source file from disk
   and runs it.
3. **Family B compiler/runner** - a separate compiler binary turns a
   `.swift` source file into `.swb` bytecode, which a runner binary
   then executes (the "two execution families" in
   [`ARCHITECTURE.md`](docs/contributing/ARCHITECTURE.md); larger programs,
   //e/Saturn only).

All paths share the same lexer, compiler, and VM. The full language
spec is in [`LANGUAGE.md`](docs/using/LANGUAGE.md). Read that before working on
anything that touches user-visible behavior.

This is a hobby/retrocomputing project. The point is partly the
destination (a working interpreter) and partly the journey (squeezing
it into 64K). Code that works but is twice as large as it needs to be
has failed half the brief.

## Read these before doing anything

1. [`CONSTRAINTS.md`](docs/contributing/CONSTRAINTS.md) - hard limits of the platform and
   toolchain. Treat as binding.
2. [`ARCHITECTURE.md`](docs/contributing/ARCHITECTURE.md) - how the system fits
   together: the pipeline, the value/memory model, and the two binary
   families. Read once for the mental model.
3. [`LANGUAGE.md`](docs/using/LANGUAGE.md) - the SwiftII language spec.
4. [`MEMORY_MAP.md`](docs/contributing/MEMORY_MAP.md) - current zero-page assignments,
   language card layout, heap layout. Update this file when you allocate
   new zero-page slots.
5. [`OPCODES.md`](docs/contributing/OPCODES.md) - current bytecode opcode list with
   semantics and operand layouts. Don't invent new opcodes without
   adding them here.
6. [`LESSONS.md`](docs/contributing/LESSONS.md) - running log of things we've learned the
   hard way (compiler bugs, surprising codegen, allocator quirks). Skim
   before starting any non-trivial change.
7. [`STYLE.md`](docs/contributing/STYLE.md) - coding style (C90 vs C17, formatting, naming,
   comments, functions). Read before writing code.

## Project layout

See [`PROJECT_LAYOUT.md`](docs/contributing/PROJECT_LAYOUT.md) for the full directory tree
and rationale. Short version: production code under `src/` is split by
module (`lexer/`, `compiler/`, `vm/`, `runtime/`, `repl/`,
`file_runner/`, `swb/` for the Family B compiler/runner, `editor/`
(compiled into the boot launcher, which itself lives under
`tools/apple2/boot_launcher/`), `platform/`, `main/`, and shared
`common/`); tests under `tests/` split by layer
(`unit/`, `sim/`, `integration/`, `repl/`) - see
[`TESTING.md`](docs/testing/TESTING.md).

## How we work

### Test-first, host-first

The default workflow is:

1. Write a failing host-side test in `tests/unit/` or
   `tests/integration/`.
2. Implement until it passes on the host (`make test`).
3. Cross-compile with cc65 (`make apple2`).
4. Run the same logic on a 6502 simulator (`make sim`) to catch
   target-specific bugs (size assumptions, signed/unsigned, etc.).
5. Only then build a disk image and try it in an emulator (`make run`).

Steps 1–3 should take seconds. Don't reach for the emulator early; it's
slow and a poor debugging environment compared to the host.

### Test both modes

When adding a feature, add tests for **both** REPL and file mode where
the behavior could differ:

- File mode: `tests/integration/NNN_feature.swift` plus
  `NNN_feature.expected`.
- REPL mode: `tests/repl/NNN_feature.repl` (a script of input lines and
  expected output lines).

Most language-feature tests should pass identically in both. The
exceptions worth watching:

- **Statement-vs-expression behavior** - the REPL implicitly
  `print()`s the value of a bare top-level expression (so `1 + 2`
  yields `3`, BASIC / Python-style); `let`/`var` declarations and
  assignments are silent. File mode discards bare-expression values
  entirely and produces no output.
- **Function redefinition** - on the //e binaries (SWIFTIIE /
  SWIFTAUX) redefining a `func` name rebinds it to the new body and
  prints a `redef foo` notice; the II+ binaries (SWIFTIIP / SWIFTSAT)
  still reject a duplicate name (they sit at the ProDOS file ceiling),
  so `:reset` is the recovery path there. See `LANGUAGE.md`.

### Small changes

A "task" should be at most a few hundred lines of diff. If a task is
bigger than that, split it. Good unit sizes:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yeokm1/swiftii](https://github.com/yeokm1/swiftii) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
