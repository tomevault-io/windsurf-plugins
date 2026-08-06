---
trigger: always_on
description: This file serves as the single point of truth for AI agents (Claude, etc.) working on the Catenary project.
---

# Catenary Agent Context

This file serves as the single point of truth for AI agents (Claude, etc.) working on the Catenary project.

## Project Grounding

- **Project Goal:** Multi-surface intelligence router — LSP-powered code intelligence for AI agents.
- **Repository:** `TwoWells/Catenary` on GitHub.
- **Config:** `@./Cargo.toml`
- **Dependency Policy:** `@./deny.toml`
- **Documentation:** `docs/src/`

## How Catenary Works

Catenary is a multi-surface intelligence router. A single daemon manages a pool
of LSP servers and exposes them through four decoupled surfaces: the **CLI**
(queries, search, and the editing lifecycle — `catenary grep`/`glob`/
`diagnostics`, invoked via the host shell tool), **hooks** (editing enforcement,
command filtering, file tracking), the **MCP server** (the protocol handshake
plus the workspace-roots channel and user-facing notifications — it advertises no
query tools; `handle_request` serves only `initialize`/`ping`), and the **TUI**
(observability). All surfaces share the same LSP server pool. None depends on the
others.

### Core concepts

- **Daemon:** A single Catenary process per host. Multiple agents connect via
  Unix domain socket. LSP servers are shared across all sessions. See
  `src/router.rs` (`SessionManager`).
- **Session:** A connected agent. Each session has a unique ID (opaque string)
  and one or more workspace roots. Live sessions appear on the `state.json`
  session board (rendered by the `catenary` TUI dashboard); their telemetry is
  queryable via `catenary query --session <id>`.
- **State & storage:** There is no primary SQLite database (a legacy
  `catenary.db` is drained on startup). State spans three XDG base dirs, chosen for
  their durability semantics (see `src/paths.rs`): the durable Unix **socket**
  under `state_dir`; the ephemeral `state.json` **session-board snapshot** under
  `runtime_dir` (tmpfs); and the
  regenerable, per-root-sharded **JSONL telemetry firehose** under `cache_dir`.
- **CLI search commands:** `catenary grep` and `catenary glob`, invoked via the
  host's shell tool. Stateless queries — no session identity, no connection
  binding. Each command connects to the daemon over a Unix domain socket,
  delegates to one or more LSP servers, and prints results to stdout.
- **Hooks:** Catenary registers several lifecycle hooks per host (Claude Code,
  Antigravity CLI); the `PreToolUse` hook handles editing enforcement,
  command filtering, and file tracking. Hook definitions (the full per-host set)
  live in
  `plugins/catenary/hooks/hooks.json` (Claude Code) and
  `plugins/catenary-antigravity/hooks.json` (Antigravity CLI).
- **CLI commands:** Editing lifecycle invoked via the host's shell tool:
  - Editing starts implicitly on the first edit — there is no explicit start
    step (`catenary editing start` remains as an idempotent no-op).
  - `catenary diagnostics` — print LSP diagnostics for the current batch of
    modified files to stdout; delivery flips each file's gate flag but the batch
    persists, so a repeat bare run re-diagnoses the same set fresh and the next
    covered edit after a fully-diagnosed batch starts a new one.
  - `catenary pin <path>` / `catenary unpin <path>` — pin or unpin a workspace
    root; bare `catenary roots` lists the current roots. Coverage is automatic —
    pinning changes a root's lifetime (stops idle expiry, pre-warms servers),
    not whether it is served.
- **Diagnostics:** `catenary diagnostics` triggers the diagnostics pipeline:
  compute over the current batch of modified files, send to LSP servers, collect
  diagnostics, print to stdout. The batch (its files, each with a delivered flag)
  lives in memory, keyed by `(session_id, agent_id)`: it persists across diagnose
  runs within a daemon instance — repeat bare runs re-diagnose the same set — and
  is **released with the instance**. On daemon death the debt is dropped, not
  spooled (maintainer ruling, bug 79): a fresh daemon disarms the gate and a bare
  run answers `[no edited files]`, so an unstable daemon never locks a session
  out. Diagnostics are recomputed fresh over the batch each run. The agent sees
  diagnostics in the shell tool output. Diagnostic events are also emitted to the
  JSONL telemetry firehose.
- **Logging:** `LoggingServer` is a `tracing_subscriber::Layer` that subscribes
  to every tracing event and dispatches structured events to its sinks: the
  per-root-sharded JSONL telemetry firehose (`src/logging/jsonl_sink.rs`, carries
  everything), the desktop-notification sink (`src/notify.rs`, error severity —
  the urgent interrupt), and the daemon `state.json` snapshot (the TUI health
  surface). The user-facing `systemMessage` notification queue retired in the TUI
  rework. See `src/logging/mod.rs` and `docs/src/tracing-conventions`.

### Key source files

- `src/paths.rs` — XDG base-dir resolvers (durable `state_dir` / ephemeral
  `runtime_dir` / regenerable `cache_dir`) and the firehose shard-key encoding.
- `src/logging/mod.rs` — `LoggingServer`: multi-sink tracing Layer, the sole
  telemetry port/adapter. Dispatches to the JSONL firehose
  (`src/logging/jsonl_sink.rs`), the desktop-notification sink, and the daemon
  snapshot writer.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TwoWells/Catenary](https://github.com/TwoWells/Catenary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
