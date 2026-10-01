---
trigger: always_on
description: Guidance for coding agents working in the Threadlane repository.
---

# AGENTS.md

Guidance for coding agents working in the Threadlane repository.

## Scope

This file applies to the entire repository.

Threadlane is a Rust workspace centered on a native GPUI desktop application (`crates/threadlane-gpui`). Keep changes focused, preserve the existing visual language, and prefer established project patterns over introducing new frameworks or dependencies.

## Repository Map

- `crates/threadlane-gpui/` — GPUI desktop application binary shell (`main.rs` + empty lib): window setup, logging, `--dump-config`, and composition of the `threadlane-ui-*` crates. Holds no product code; add new UI to a focused leaf crate instead.
- `crates/threadlane-ui-workspace/` — window root (`WorkspaceView`): composes chat, sidebar, panels, terminal, settings, and GitHub over one `AppState`; owns global keybindings and background event pumps.
- `crates/threadlane-ui-theme/` — shared GPUI visual language: theme registry wiring, design tokens (`WINDOW_CONTROLS_CLEARANCE`, `overlay_scrim()`), bundled `themes/threadlane.json`, and vendored `assets/icons/`. Depends only on GPUI/kit plus `threadlane-project` for the global dir; every screen crate uses it, never the reverse.
- `crates/threadlane-daemon/` — headless, GPUI-free service core a future daemon process wraps: session discovery/projection, chat turn services (`chat`: prompt execution, titles, ACP config), the agent-event adapter, background services (`provider_auth`, `settings`, `updater`), worktree setup, the automation application service, the `next_event_batch` stream helper, and the picker-facing model catalog (`catalog`: provider inventory, live discovery caches, effort metadata — the four `*_and_update` refresh wrappers that need `Entity<AppState>` live in `threadlane-ui-settings`, which the workspace shell imports directly). `threadlane-ui-state` re-exports all of it for compatibility.
- `crates/threadlane-ui-sidebar/` — project/session sidebar plus the shared keybindable `BeginNewTask` action (owned here because the sidebar's buttons dispatch it; the workspace shell imports the type for its cmd-N binding).
- `crates/threadlane-ui-state/` — durable desktop UI state, GPUI-free: `AppState`, app intent (`actions`/`controller`), and UI-only types (workspace page, composer/insert requests, GitHub tab). Session services moved to `threadlane-daemon` and are re-exported here (`chat`, `discovery`, `projection`, `automation`, `provider_auth`, `settings`, `updater`, `worktree_setup`, `agent_events`, `catalog`, session/projection types) so existing `threadlane_ui_state::*` paths keep working; file watching lives in `threadlane-project::watcher`.
- `crates/threadlane-ui-terminal/` — self-contained PTY terminal view (`TerminalView::new(PathBuf, cx)`): portable-pty session, frame-budgeted parsing, and GPUI rendering. No `threadlane-*` or `AppState` dependency; the workspace screen constructs it directly.
- `crates/threadlane-ui-chat/` — chat surface (`ChatListView`, `CentralTab`, composer/transcript/trajectory/context-meter): depends on `threadlane-ui-state`, `threadlane-daemon` (model catalog), `threadlane-ui-theme`, plus `threadlane-ui-editor` and `threadlane-ui-mirror` which it embeds.
- `crates/threadlane-ui-editor/` — file editor view (`EditorView`, `SaveFile` action); depends only on `threadlane-ui-state` (+`rfd`, `tracing`).
- `crates/threadlane-ui-mirror/` — computer-use live mirror (`MirrorView` over `threadlane-protocol::live` frames); depends only on `threadlane-ui-state`. NOTE: never use `use super::*` in its `#[cfg(test)]` module — the glob pulls gpui macros into test scope and `cargo check --tests` hangs in macro expansion (needs `recursion_limit >= 2048`, and even that hangs). Explicit test imports compile in ~2s at the default limit.
- `crates/threadlane-ui-settings/` — settings screen plus the model-catalog `*_and_update` refresh wrappers (they need `Entity<AppState>`, so they cannot live in GPUI-free `threadlane-daemon`).
- `crates/threadlane-ui-github/` — GitHub issues/PRs/diffs/review timelines; `view::init` registers its actions.
- `crates/threadlane-ui-right-panel/` — git review, draft PRs, embedded browser (macOS `gpui-wry` view, stub elsewhere), and file surfaces. `retain_review_selection` is specified by `tests.rs`; keep them in sync.
- `crates/threadlane-runtime/` — agent execution engine, harness V2 durability, provider routing, compaction, and turn driver. Depends on `threadlane-provider`, never the reverse.
- `crates/threadlane-coding-agent/` — coding-agent engine: `CodingAgent` runtime, durable `harness`, subagents, scheduler, mailbox, capabilities, `controller` (`SessionController`; also home of the `SessionRuntime` alias, `runtime_status_text`, `spawn_session_runtime_construction`, and the `test-support` provider-injection helper consumed by the GPUI crates), `config_dump`, the `acp_stub_agent` test fixture plus engine integration `tests/`, and the session-owned adapters it is built from (`commands`, `computer`, `credentials`, `mcp`). The `threadlane-gpui` binary calls `config_dump` and owns `process_environment` directly. There is no separate `threadlane-session` crate; UI crates import `threadlane_coding_agent::controller` directly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wheregmis/threadlane](https://github.com/wheregmis/threadlane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
