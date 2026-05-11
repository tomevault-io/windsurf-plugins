---
trigger: always_on
description: Writer is a Tauri v2 desktop markdown editor: React frontend + Rust backend. It targets writers who use local-first plain-text workflows (Obsidian vaults, docs repos, personal wikis).
---


Writer is a Tauri v2 desktop markdown editor: React frontend + Rust backend. It targets writers who use local-first plain-text workflows (Obsidian vaults, docs repos, personal wikis).

## This File

This is an **agent router**: concise context loaded every session. It routes the agent to relevant docs based on task intent. Deep guidelines, system rules, and review rubrics live in linked docs — not here.

## Architecture (Brief)

Stack: Tauri v2 (React + Rust)
Frontend: `apps/desktop/src/` — React, Zustand stores, CodeMirror/Prosemark editor
Backend: `apps/desktop/src-tauri/src/` — Rust IPC commands, file watcher, workspace state
Toolchain: Vite+ (`vp`) — see [docs/vite-plus.md](./docs/vite-plus.md)

Rust source structure:

- `lib.rs` — app setup, plugin registration, command registration
- `state.rs` — global app state (workspace root, file index, watcher handle)
- `error.rs` — error types serialized over IPC
- `watcher.rs` — file system watcher with debounce and self-write detection
- `commands/` — Tauri IPC handlers: `fs.rs`, `workspace.rs`, `search.rs`, `images.rs`

## Docs Index

All docs except CLAUDE.md, AGENTS.md, TODOS.md, and CHANGELOG.md live in `./docs/`. Feature specs live in `./SPECs/`.

**Workflow**

- [docs/workflows/agent-loop.md](./docs/workflows/agent-loop.md) — repeatable autonomous task execution loop
- [docs/workflows/agent-review.md](./docs/workflows/agent-review.md) — review personas, findings format, quality checklist, escalation rules
- [docs/workflows/worktrees.md](./docs/workflows/worktrees.md) — creating and managing parallel worktrees

**Guidelines**

- [docs/consolidation.md](./docs/consolidation.md) — if adding the next case touches more than one file, the structure is wrong: single source of truth, side-effect ownership, registry over per-case branches, one write path
- [docs/react-guidelines.md](./docs/react-guidelines.md) — imports, state, side effects, component structure, persistence
- [docs/zustand.md](./docs/zustand.md) — side effect timing, selectors, bail-out patterns
- [docs/vite-plus.md](./docs/vite-plus.md) — `vp` CLI usage and common pitfalls
- [docs/keyboard-shortcuts.md](./docs/keyboard-shortcuts.md) — canonical shortcut map

**Infra**

- [docs/releasing.md](./docs/releasing.md) — how to cut a signed, notarized macOS release
- [docs/website-deploy.md](./docs/website-deploy.md) — how to deploy the marketing website to Cloudflare Workers

**Cross-cutting**

- [TODOS.md](./TODOS.md) — task backlog and work-in-progress tracking
- [CHANGELOG.md](./CHANGELOG.md) — user-visible changes log
- [SPECs/](./SPECs/) — human-written feature and bug specs

## Hard Rules

- Check [`TODOS.md`](./TODOS.md) before starting work to see current tasks.
- Move tasks between sections (Up Next → In Progress → Done) as you work.
- For non-trivial work, create a spec in [`SPECs/`](./SPECs/) and link it from the task.
- Update [`CHANGELOG.md`](./CHANGELOG.md) when completing work.
- Load only docs relevant to the current task and scope.
- If a behavior or rule changes in practice, update the owning doc in the same task.
- In autonomous/loop mode, complete exactly one task at a time and commit immediately. Do not batch unless the user explicitly asks.
- When coding or reviewing, follow the Engineering Guardrails below and the guidelines in linked docs.

## Engineering Guardrails

- Prefer the smallest change that is correct, robust, and easy to reason about.
- Avoid fragile logic, hidden coupling, edge-case traps, and assumptions about incidental execution order.
- Keep async flows and shared state race-safe. Use explicit sequencing, cancellation, idempotency, or clear ownership where needed.
- Do not introduce unnecessary performance regressions. Avoid extra renders, allocations, subscriptions, scans, blocking work, or I/O unless clearly justified.
- Fail explicitly. Surface invalid states and unexpected errors clearly instead of silently swallowing them or masking them with fallback behavior.
- Preserve testability. Keep side effects at the boundaries, make dependencies explicit, and structure logic so it can be exercised in isolation when practical.
- Maintain clear boundaries, but prefer minimal designs. Split functions, hooks, or modules when responsibilities meaningfully diverge, not as a reflex.
- If a tradeoff is unavoidable, call it out explicitly and choose the option with the lowest long-term correctness and maintenance risk.

## Validation

Frontend:

- `vp check` — format, lint, and TypeScript type checks
- `vp test` — JavaScript/TypeScript tests

Rust (from `apps/desktop/src-tauri/`):

- `cargo test`
- `cargo clippy`
- `cargo fmt --check`

## Session Wrap

Wrap per [agent-loop.md](./docs/workflows/agent-loop.md). One commit per completed task with a clear message. See the existing commit history for style.

<!--VITE PLUS START-->

# Using Vite+, the Unified Toolchain for the Web


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joelbqz/writer-computer](https://github.com/joelbqz/writer-computer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
