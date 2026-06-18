---
trigger: always_on
description: This document records RARA goals, scope, architecture constraints, and documentation rules.
---

# RARA Project Charter

This document records RARA goals, scope, architecture constraints, and documentation rules.
It is the baseline index for future implementation and evolution.

---

## Index

### Project
- [1. Project Goals](#1-project-goals)
- [2. Scope](#2-scope)
- [3. Architecture Constraints](#3-architecture-constraints)
- [4. Current Key Decisions](#4-current-key-decisions)
- [5. Documentation Rules](#5-documentation-rules)
- [6. Near-Term Focus](#6-near-term-focus)
- [7. Repository Skills](#7-repository-skills)
- [8. Commit Rules](#8-commit-rules)

### Docs
- [docs/features/](docs/features/README.md) — engineering specs and contracts (28 files)
- [docs/journal/](docs/journal/) — dated implementation notes and checkpoints
- [docs/journal/2026-05-07-file-split-lessons.md](docs/journal/2026-05-07-file-split-lessons.md) — file splitting patterns, pitfalls, and workflow
- [docs/todo.md](docs/todo.md) — active follow-up work

---

## 1. Project Goals

RARA is a local-first coding agent runtime with:

- a terminal chat and TUI surface;
- pluggable LLM backends;
- an agent loop that can call tools and continue after tool results;
- durable local memory and workspace context;
- room for both hosted providers and local model execution.

The current product direction is to make local inference a first-class path instead of a fallback.

## 2. Scope

- Interactive TUI chat flow.
- Tool-calling agent loop.
- Local workspace and project memory.
- Hosted-provider integration where useful.
- Local model execution through Candle-backed runtimes.

## 3. Architecture Constraints

- Backend/runtime language: Rust.
- The primary execution surface is a local CLI/TUI binary.
- The agent loop should continue to depend on a stable backend trait instead of model-specific code paths.
- Local models should plug into the same `LlmBackend` contract used by hosted providers.
- TUI interaction should converge toward one unified prompt surface instead of growing separate setup-only flows for common actions.
- Prefer smaller modules over long files; as a rule of thumb, avoid letting a single source file grow beyond roughly 800 lines unless there is a strong reason not to split it.
- If an implementation would push a source file toward or past that limit, proactively split the file instead of continuing to accumulate new logic in place.
- Non-trivial behavior changes should add or update focused tests when practical.
- Before implementing any non-trivial behavior change, first inspect the relevant Codex and Claude Code implementations, extract the interaction or runtime pattern that applies, write a short plan for how RARA should mirror or adapt it, and only then start implementation.

## 3.1 Rust Engineering Rules

- Always run `cargo fmt` to maintain consistent code style.
- New or modified Rust code must not introduce new compiler or Clippy warnings.
  The existing codebase may still contain legacy warnings; keep changes
  warning-clean within the touched scope, and prefer fixing nearby warnings
  only when they are directly caused by or blocking the current change.
- Avoid ambiguous positional booleans, numeric literals, or `Option` arguments in new APIs when they make call sites hard to read. Prefer enums, newtypes, named methods, or small parameter structs.
- Prefer exhaustive `match` statements over wildcard arms when the variants are part of a meaningful state machine or protocol contract.
- Newly introduced traits should include concise doc comments that explain the trait role and what implementors must preserve.
- Keep modules private by default and expose public APIs intentionally through explicit module exports.
- Do not add helper functions that are only used once unless they name a non-obvious invariant or isolate a testable boundary.
- When adding a new concept, first check whether it belongs in an existing narrow crate/module or whether a small new module avoids growing a high-touch orchestration file.
- Avoid `unsafe` unless it is strictly necessary. When `unsafe` is required,
  isolate the smallest possible boundary and add a concise `// SAFETY:`
  comment explaining the invariant that makes the block sound.

- **Errors must surface.** Do not silently ignore errors. Use `log::warn!` for
  non-fatal errors so they appear in the TUI alongside the conversation. A silent `let _ = fallible_call();` or `.ok()` discard is
  only acceptable when the error carries zero diagnostic value AND the
  caller has no recovery path. When in doubt, log it.

- Dead code is not permitted in production source files. Use `#[cfg(test)]`
  for test-only helpers; remove all unreachable types, functions, and constants.
- `#![allow(dead_code)]` at module level is not permitted — it silently
  hides real dead code.  Every intentionally-unused item must carry its own
  `#[allow(dead_code)]` with a comment explaining why and when it activates.
- Adding `#[allow(dead_code)]` to an individual item requires a comment
  explaining why the item is intentionally unused and when it will be activated.
- File-size violations detected in review (source files exceeding 800 lines
  under `src/` or `crates/`) must be fixed before merge, not deferred to a
  follow-up task.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linkerdog/rara](https://github.com/linkerdog/rara) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
