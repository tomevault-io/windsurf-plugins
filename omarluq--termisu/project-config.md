---
trigger: always_on
description: Guidance for coding agents working in the Termisu repository.
---

# AGENTS.md

Guidance for coding agents working in the Termisu repository.

## Scope

- This repo is primarily a Crystal library for terminal UIs.
- There are also Bun/TypeScript workspaces in `javascript/core` and `e2e`.
- Prefer changing the smallest surface area that solves the task.
- Follow existing architecture and naming before introducing new abstractions.
- Read `CLAUDE.md` first for architecture, command aliases, and project-specific patterns.
- Check `CLAUDE.md` for any repo-shipped agent guidance beyond this file.
- Check `PROJECT_INDEX.md` for a compact project overview.
- Look at `spec/` and `examples/` before inventing new APIs or behaviors.

## Instruction Files Present

- `CLAUDE.md` exists at the repo root and contains repo-specific guidance.
- No repo-local `.claude/` instruction directory was found in this branch.
- No `.cursor/rules/` directory was found.
- No `.cursorrules` file was found.
- No `.github/copilot-instructions.md` file was found.

## Setup

```bash
shards install
shards build ameba
shards build hace
```

## Core Commands

```bash
# Crystal library
bin/hace spec
bin/hace format
bin/hace format:check
bin/hace ameba
bin/hace ameba:fix
bin/hace all

# Single Crystal spec file
crystal spec spec/termisu/buffer_spec.cr

# Single Crystal spec example by line number
crystal spec spec/termisu/buffer_spec.cr:149

# Single Crystal spec by example name
crystal spec spec/termisu/buffer_spec.cr --example "forces full re-render"

# Examples
bin/hace demo
bin/hace showcase
bin/hace animation
bin/hace colors
bin/hace kmd

# C ABI
bin/hace ffi:build
bin/hace c:test
bin/hace c:check

# JS / E2E
bun install
bin/hace js:typecheck
bin/hace js:test
bin/hace js:check
bin/hace e2e:test
bin/hace e2e:check

# Docs site
bin/hace docs:build
bin/hace docs:serve
```

## Single-Test Guidance

- For Crystal, prefer `crystal spec path/to/file_spec.cr` for a single file.
- To run one example, use `crystal spec path/to/file_spec.cr:<line>`.
- `--example` also works when the example name is stable.
- Crystal specs mirror `src/` structure under `spec/termisu/`.
- For Bun tests in `javascript/core`, run `bun test tests/path/to/file.test.ts` from `javascript/core`.
- For E2E tests, inspect the `e2e` workspace and run the narrowest supported `tui-test` target if the test runner accepts one; otherwise use the full `bin/hace e2e:test` command.

## Validation Order

- For Crystal-only changes: run `bin/hace format`, `bin/hace ameba`, then `bin/hace spec`.
- For JS workspace changes: run the relevant `bin/hace js:*` or `bin/hace e2e:*` commands.
- Before finishing a non-trivial change, prefer `bin/hace all` if the touched areas are covered by it.
- If you touch C ABI files, also run `bin/hace c:check` and `bin/hace c:test`.

## Repository Layout

- `src/termisu/` contains the main Crystal implementation.
- `src/termisu.cr` is the public entry point and requires the internal tree.
- `spec/termisu/` mirrors the `src/termisu/` structure.
- `spec/support/`, `spec/shared/`, and `examples/` are the best behavior references.
- `javascript/core/` contains Bun FFI bindings; `e2e/` contains terminal integration tests.

## Subsystem Maintainer Map

- The repo currently has a single primary maintainer: [omarluq](https://github.com/omarluq). Use the map below as routing guidance for where to look first, which files usually move together, and which changes deserve extra care before handing work back.
- Public Facade: maintainer `@omarluq`; start in `src/termisu.cr`, `src/termisu/terminal.cr`, and top-level public types such as `src/termisu/color.cr` and `src/termisu/attribute.cr`.
- Terminal State + Low-Level I/O: maintainer `@omarluq`; inspect `src/termisu/terminal/`, `src/termisu/tty.cr`, and `src/termisu/termios.cr` together because mode changes, cleanup, and fd ownership are tightly coupled.
- Rendering Core: maintainer `@omarluq`; changes usually span `src/termisu/buffer.cr`, `src/termisu/render_state.cr`, `src/termisu/cell.cr`, and cursor/color state.
- Input Reading: maintainer `@omarluq`; start with `src/termisu/reader.cr` and adjacent terminal backend code before changing buffering, blocking, or read-loop behavior.
- Input Parsing: maintainer `@omarluq`; focus on `src/termisu/input/parser.cr`, `src/termisu/input/key.cr`, and `src/termisu/input/modifier.cr`; confirm behavior against existing parser specs before changing escape handling.
- Event System: maintainer `@omarluq`; read `src/termisu/event/loop.cr`, `src/termisu/event.cr`, `src/termisu/event/source*.cr`, and event payload types as one subsystem.
- Poller Backends / Cross-Platform Timing: maintainer `@omarluq`; inspect `src/termisu/event/poller/`, `src/termisu/event/source/timer.cr`, and `src/termisu/event/source/system_timer.cr`; preserve platform branches and shutdown semantics.
- Terminfo + Capability Layer: maintainer `@omarluq`; start in `src/termisu/terminfo/`; parser, database lookup, builtins, and `tparm` logic should be treated as one compatibility surface.
- Logging: maintainer `@omarluq`; keep `src/termisu/log.cr` low-noise and best-effort, and avoid introducing logging that changes terminal timing or cleanup paths.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omarluq/termisu](https://github.com/omarluq/termisu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
