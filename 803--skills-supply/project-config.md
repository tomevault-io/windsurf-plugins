---
trigger: always_on
description: > **Note**: This project uses [bd (beads)](https://github.com/steveyegge/beads) for issue tracking. Use `bd` commands instead of markdown TODOs. See the Issue Tracking section below for workflow details.
---

# AGENTS.md

> **Note**: This project uses [bd (beads)](https://github.com/steveyegge/beads) for issue tracking. Use `bd` commands instead of markdown TODOs. See the Issue Tracking section below for workflow details.

> **Purpose**: The bare‑minimum rules this repo cares about. Follow exactly. When unsure, ask.

---

## ExecPlans

When writing complex features or significant refactors, use an ExecPlan (as described in .agent/PLANS.md) from design to implementation.

---

## Runtime & Dev Environment

* **Browser access**: You should have access to the browser.
* **DB**: Postgres is already running **in Docker**. Use containerized access.
  * **psql**: Not installed on host. Run inside Docker (`docker-compose.yaml` has details / service names).
* **Server**: Already running; when launched via `./watchexec.sh`, logs are tee'd to `./.logs/<timestamp>.log`.
* **Hot reload**:
  * Changes to `./config` or `./lib` **automatically restart the server**
  * Changes to `./assets` or `./packages` **automatically rebuild static assets**.
  * `./watchexec.sh` runs the dev server under `watchexec` to enforce the above:
    - Restarts on `./config` changes (Phoenix requires a hard restart for config edits) and also watches `./lib`.
    - Streams stdout/stderr to the terminal and tees to `./.logs/<timestamp>.log`.
    - Run with: `./watchexec.sh` (requires `watchexec`).

---

## Core Principles

* **Self‑documenting code; comments are for _why_** (rationale, invariants, tradeoffs). Do not narrate what code already states.
* **Functional core, imperative shell**.
* **Fail loudly on ambiguity**: if input isn’t explicitly handled, error fast. Prefer explicit ignores over silent catch‑alls.
* **Idiomatic over clever**.
* **No in‑code TODO sprawl**: Do **not** add `TODO` comments in source files. Use **bd (beads)** to track tasks
* **Consistent error shapes**: Within a module/service, keep the exact error/ok/result shape consistent (e.g., always `{:ok, v} | {:error, e} | {:state, v}` or a single Result type). Only `raise` when the process **should die**.
* **No imports inside function bodies**: Never put `import`/`alias`/`require` (or language‑equivalents) inside functions/blocks. Imports go at the file/module top.
* **Data structures first**: Design types/interfaces before logic. Good data models make code obvious; complex algorithms often mean wrong structures. "Show me your tables and I won't need your flowcharts."
* **Explicit boundaries**: Know where system edges are (CLI input, file I/O, database, network, user display). Transform data **once** at boundaries, not throughout the system.
* **Rich types over primitives**: Keep objects/typed values until the last moment. Convert to strings/derived values only at display or serialization boundaries.
* **Composability over configuration**: Build small, explicit functions that compose. Implicit configuration ("this coerces if X, does Y if Z") hides intent and makes testing harder.

---

## Error Handling Strategy

* **Result types at core boundaries**: Pure/core modules return Result-like types (`{ok, value} | {ok, error}`) to make error paths explicit and composable.
* **Throw only at application edges**: CLI/web layers unwrap Results and throw for user-facing errors. Core logic never throws.
* **Rich error shapes**: Errors carry context (type, message, path/field, original cause). Don't lose information as exceptions bubble up.
* **Fail fast, fail loudly**: When input can't be handled, return error immediately. Don't default, don't silently ignore. Ambiguity is a bug.

When in doubt: If you're catching exceptions in core code to return them, you're in the wrong place. Push error handling to the boundary.

---

## Interface Design

* **Small, focused units**: Each function does one thing well. 3-5 parameters max. If you need more, introduce a parameter object.
* **Explicit over implicit**: Caller decides composition (`coerceThenParse()` vs `parse()` that secretly coerces). Magic in function bodies is technical debt.
* **Validated types guarantee invariants**: Use branded types or similar mechanisms to ensure data passed through validation can't become invalid downstream.
* **Separation of declaration and validated types**: `RawDependency` vs `ValidatedDependency`. Transform at boundaries, keep core simple.

Red flag: Function signature has `boolean` or `string` flags to control behavior ("if `force`, do X; if `quiet`, do Y"). Prefer separate functions or parameter objects with descriptive names.

---

## State Management

* **State is explicit, not implicit**: State files/state machines track what happened. Don't infer state from filesystem alone.
* **Reconcile, don't overwrite**: Sync means "make installed match manifest". Compute diff, apply changes atomically. `npm prune` pattern, not `rm -rf`.
* **Track what you manage**: State files list what your tool installed. Don't delete manually-added skills. Reconcile only removes tracked items.
* **State transitions are explicit**: Functions like `read → diff → reconcile → write` make flow clear. One function doing all three is a smell.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [803/skills-supply](https://github.com/803/skills-supply) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
