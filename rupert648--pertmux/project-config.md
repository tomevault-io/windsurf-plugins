---
trigger: always_on
description: This document provides a technical overview of the pertmux codebase for AI agents and developers.
---

# pertmux: Agent Guide

This document provides a technical overview of the pertmux codebase for AI agents and developers.

## Project Overview
pertmux is a Rust TUI unified SWE dashboard that links GitLab/GitHub MRs to local branches/worktrees, tmux sessions, and coding agent instances. It provides a real-time view of session status, resource usage, and progress with integrated merge request tracking across multiple forges. The bottom panel provides worktrunk-powered worktree management with create/remove/merge actions. The architecture is pluggable — new coding agents can be added by implementing the `CodingAgent` trait, and new forges can be added by implementing the `ForgeClient` trait.

## Architecture
The project uses a **daemon/client architecture** with Unix socket IPC. A background daemon (`pertmux serve`) owns all data fetching and state, while a lightweight TUI client (`pertmux connect`) connects to render the UI.

### Daemon/Client Split
- **Daemon** (`daemon.rs`): Runs persistently in background. Owns the `App` struct (which is not `Send` due to `dyn CodingAgent`), runs on the main tokio task. Performs all data fetching on configurable timers: tmux/agent (`refresh_interval`, default 2s), MR detail (`mr_detail_interval`, default 60s), worktrees (`worktree_interval`, default 30s), MR list (`mr_list_interval`, default 300s). Listens on `/tmp/pertmux-{USER}.sock`.
- **Client** (`client.rs`): Lightweight TUI. Owns all UI state (`ClientState`: selection indices, popup state, notifications). Connects to daemon via Unix socket, receives `DashboardSnapshot` updates, sends commands (`Refresh`, `CreateWorktree`, etc.). Navigation is instant with no daemon round-trip.
- **Protocol** (`protocol.rs`): Defines `DashboardSnapshot`, `ProjectSnapshot`, `ClientMsg`, `DaemonMsg`. Framed with `LengthDelimitedCodec` + `serde_json`. Multi-client via `tokio::sync::broadcast`.

### Data Flow
1. **Daemon startup**: Loads config, validates projects, creates `App`, performs initial fetch of MRs + tmux + worktrees.
2. **Refresh loops**: Daemon runs configurable tiered timers (`refresh_interval` default 2s tmux, `mr_detail_interval` default 60s MR detail, `worktree_interval` default 30s worktrees, `mr_list_interval` default 300s MR list). After each refresh, broadcasts `DashboardSnapshot` to all connected clients.
3. **Client connect**: Connects to daemon socket. Fails with clear error if daemon not running. Receives initial snapshot immediately.
4. **Client commands**: User actions (refresh, worktree create/remove/merge, MR selection) are sent as `ClientMsg` to daemon. Daemon processes, refreshes relevant data, broadcasts updated snapshot.
5. **tmux actions**: `switch_to_pane()` and `find_or_create_pane()` run client-side — they only need data from the snapshot, not daemon state.

## Module Guide
- **main.rs**: Entry point. Uses clap for subcommands: `serve` → `daemonize()` (or `daemon::run()` with `--foreground`), `connect` → `client::run()`, `stop` → `client::stop()`, `status` → `client::status()`, `cleanup` → `client::cleanup()`. `serve` self-daemonizes by default: re-execs with `--foreground`, redirecting stdout/stderr to `/tmp/pertmux-daemon.log`, detached via `process_group(0)`. Validates config and checks for existing daemon before forking. Requires explicit subcommand (no bare `pertmux`).
- **daemon.rs**: Background daemon. Unix socket listener with `LengthDelimitedCodec` framing. Broadcast channel for multi-client snapshot fan-out. `Arc<Mutex<DashboardSnapshot>>` for latest snapshot (sent to new clients immediately). Handles `ClientMsg` commands and runs tiered refresh intervals. Tracks client count via `Arc<AtomicUsize>` and accumulates MR changes in `pending_for_offline` when no clients are connected — drains them into the initial snapshot on reconnect.
- **client.rs**: TUI client. Connects to daemon (fails with error screen if not running), owns `ClientState` with all UI state (selections, popup, notification). Event loop with `tokio::select!` on keyboard + daemon messages. Local navigation (j/k/Tab) with no round-trip. Project switching via fuzzy finder (`f` key). Also provides `stop()`, `status()`, and `cleanup()` commands. On reconnect, if `pending_changes` is non-empty, opens `ChangeSummary` modal; for live snapshots, shows toast notifications.
- **protocol.rs**: IPC protocol. Defines `DashboardSnapshot`, `ProjectSnapshot`, `GlobalMrEntry` (cross-project MR entries), `ClientMsg` (commands from client to daemon), `DaemonMsg` (responses/snapshots from daemon to client), `PROTOCOL_VERSION` for handshake validation. Also defines `ActivityEntry` (feed items), `ActivityKind` (display colour), and `ActivityTarget` (navigation destination — `Pane { pane_id, pane_path }` for agent events, `MergeRequest { project_name, iid }` for forge events). The `target` field is `#[serde(default)]` for backwards compatibility.
- **app.rs**: Owns the `App` struct, which holds data state (panes, projects, MRs, worktrees). Manages refresh cycle, linking, and `snapshot()` method to produce `DashboardSnapshot`. UI-related methods (selection, popup) have moved to `ClientState` in `client.rs`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rupert648/pertmux](https://github.com/rupert648/pertmux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
