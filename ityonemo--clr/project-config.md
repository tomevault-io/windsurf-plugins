---
trigger: always_on
description: CLR is a Zig-based static analysis plugin for a forked Zig compiler. The plugin
---

# AGENTS.md

## Project Overview

CLR is a Zig-based static analysis plugin for a forked Zig compiler. The plugin
emits `.air.zig` analyzer programs from Zig AIR, and the runtime library in
`lib/` executes those analyzers to detect memory safety, undefined-value, null,
variant, fieldParentPtr, and file-descriptor issues.

## Important Directories

- `src/`: compiler plugin and AIR-to-Zig code generation.
- `lib/`: runtime analysis library used by generated analyzers.
- `lib/analysis/`: individual safety analyses and their focused tests.
- `test/cases/`: Zig input programs used by integration tests.
- `test/integration/`: BATS integration tests.
- `zig/`: forked Zig compiler submodule. Do not modify unless explicitly asked.
- `vendor/`: vendored validation projects and external inputs.

## Build And Test Commands

- Build plugin: `zig build -Doptimize=ReleaseFast`
- Codegen/plugin tests: `zig build test`
- Runtime library tests: `zig test lib/lib.zig`
- Single test case: `./run_one.sh test/cases/path/to/case.zig`
- Integration test file: `bats test/integration/name.bats`
- Full integration suite: `./run_integration.sh`
- Clean generated analyzer files: `./clear.sh`

Use `ReleaseFast` by default. The vendored Zig compiler and `libclr.so` must be
built with matching optimization modes or plugin loading may crash.

## Development Workflow

For bug fixes and new analysis behavior:

1. Add or identify a failing test before changing implementation code.
2. Prefer a focused unit test in `lib/analysis/*_test.zig` or
   `src/codegen_test.zig` when the bug is localized.
3. Add a minimal Zig case under `test/cases/`.
4. Add BATS coverage under the appropriate `test/integration/*.bats` file.
5. Fix the implementation in `lib/`, `src/`, or both.
6. Run the narrowest relevant tests first.
7. Run the full integration suite once before committing broad changes.
8. Document completed fixes in `FIXES_LOG.md`.

Do not delete, skip, or weaken integration tests to make the suite pass without
explicit permission. Investigate failures instead of assuming they are
pre-existing.

## Debugging Generated Analyzers

`./run_one.sh` generates a root-level `.air.zig` file. After generation, run it
directly for faster iteration:

```sh
zig run --dep clr -Mroot=name.air.zig -Mclr=lib/lib.zig
```

It is acceptable to temporarily instrument generated `.air.zig` files while
debugging. They are ignored by git and can be removed with `./clear.sh`.

For raw AIR around a function, use:

```sh
./dump_air.sh test/cases/path/to/case.zig function.name 80
```

Do not use `-femit-air` or `--verbose-air` directly; use the helper scripts.

## Coding Guidelines

- Match existing Zig style and local helper APIs.
- Keep changes scoped to the affected analysis or codegen path.
- Avoid broad refactors while fixing a specific false positive or false
  negative.
- Commit messages should indicate the commit content contains code written by
  Codex.
- Treat `.air.zig` files, `.zig-cache/`, and `zig-out/` as generated artifacts.
- Do not modify `zig/` unless the task explicitly requires compiler changes.
- Do not change runtime data-structure types in `lib/` without asking first.
  This includes adding fields or union/enum cases to analysis state,
  `Refinements` entities, `Analyte`, `Inst`, or other shared runtime structures.
  Prefer using existing state representations and stdlib boundary overrides when
  they fit the compiler-enforced Zig type shape.
- Be careful with dirty worktrees; preserve user changes and avoid reverting
  unrelated files.

## Memory Safety Model

Zig-CLR memory analysis is provenance and safety tracking, not Rust-style
single-owner transfer. More than one value or code path may refer to the same
allocation root GID. When any path marks that allocation freed, the ambiguous
safety state collapses around that root and later access/free/leak checks should
use that state. Aliasing will be tracked separately.

- Treat the allocation root GID as the allocation identity.
- Use `allocator_gid` only for allocator mismatch detection.
- Do not model allocation responsibility as exclusive ownership transfer.
- Do not copy a pointer value's `.to` target across `Refinements` tables.
  Cross-table operations may merge/import structure, but pointer values should
  point only at GIDs that already exist in their destination table.
- `ptr_add` and `ptr_sub` both produce derived pointers into the same region.
  Neither operation proves that a pointer has returned to the allocation base.
  Freeing memory through such a derived pointer should remain invalid unless an
  explicit future retag feature, or an internal CLR stdlib override for a known
  Zig stdlib pattern, reestablishes base-allocation provenance. Pointer
  arithmetic must operate on a pointer-to-region; non-pointer or single-item
  pointer inputs should produce a clear analysis error.

## Test Expectations

`zig build test` only covers `src/` tests. When changing runtime analysis logic,
also run `zig test lib/lib.zig`.

Integration tests should assert exit status, error type, function name, source
location, and relevant context messages where applicable.

## Current Baseline And Risks

The integration baseline after allocator provenance, call-return, test

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ityonemo/clr](https://github.com/ityonemo/clr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
