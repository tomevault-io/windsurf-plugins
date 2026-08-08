---
trigger: always_on
description: Greenfield, Apache-2.0, macOS-only Rust workspace. Read `PLAN.md` before
---

# AutoHarness — Agent Guide

Greenfield, Apache-2.0, macOS-only Rust workspace. Read `PLAN.md` before
changing contracts. Phase work ends in working software with a focused
commit per phase.

## Conventions

- Rust edition 2024 (workspace `resolver = "3"`), stable toolchain.
- Library crates under `crates/`, binaries under `bins/`.
- Keep dependencies minimal; the policy list is in the root `Cargo.toml`
  workspace table. Confirm a crate is already depended upon before using it.
- No `unsafe` outside the daemon's `getpeereid` peer-UID check,
  `engines::process` (pre_exec setrlimit, killpg), and the menu-bar's single
  fixed AppKit target/action selector. Each boundary is documented beside the
  block; no selector or executable input may come from a user or model.
- Never panic on bad input: illegal state transitions, malformed frames, and
  unknown RPCs return structured errors.
- One commit per phase; do not mix phases in a commit.

## Crate map

| Crate | Path | Owns |
|-------|------|------|
| `autoharness-core` | `crates/core` | Domain types (EngineKind, RunState, NodeState, RouteDecision, GraphProposal, Budget, Artifact, Checkpoint, MemoryFact, PolicyVersion); run/node state machines as `Result`-returning functions; `router` (task profiling, route selection, budgets) and `detector` (fingerprints, loop/stall triggers, recovery ladder) — both pure |
| `autoharness-protocol` | `crates/protocol` | JSON-RPC 2.0 envelopes (`Request`/`Response`/`Event`), 4-byte length-delimited framing, method constants, `DedupCache` |
| `autoharness-store` | `crates/store` | SQLite (rusqlite, bundled, WAL), migrations table, append-only event ledger, checkpoints/snapshots, replay, engine-session persistence |
| `autoharness-daemon` | `crates/daemon` | Unix socket server (0600, `getpeereid` UID check), token auth, RPC dispatch, `events.subscribe` replay+live, run lifecycle, `runner.rs` (direct + bounded-loop run task, steering, detector/recovery, finalize, restart reconciliation), `worktree.rs` (daemon-managed git worktrees), `sandbox/` (Seatbelt runner, profiles, proxy, canaries, policy) |
| `autoharness-engines` | `crates/engines` | `EngineAdapter` contract, normalized `EngineEvent` model, `EngineDiagnostics`, FakeEngine, Codex (app-server v2) and Claude (stream-json) adapters, contract suite, `EngineRegistry` |
| `autoharness-ui-gpui` | `crates/ui-gpui` | GPUI desktop shell. See the module map below. |
| `autoharness` | `bins/autoharness` | Desktop binary |
| `autoharnessd` | `bins/autoharnessd` | Daemon binary |
| `autoharness-updater` | `bins/updater` | Authenticated update request, independent bundle verification, active-run refusal, atomic swap, rollback, and relaunch |

## Shell module map (`crates/ui-gpui/src`)

| Module | Owns |
|--------|------|
| `lib.rs` | The `Shell` root view, layout, key routing, the utility overlays, and the in-app attention banner |
| `client.rs` | Daemon socket client, `UiState`, command encoding, response and event folding |
| `theme.rs` | Design tokens: type scale, radii, spacing, colours, fills, status vocabulary, motion |
| `layout.rs` | Pane widths and heights, seams, minimum sizes |
| `toolbar.rs` `sidebar.rs` `coordinator.rs` `workbench.rs` `inspector.rs` | The panes of the cockpit |
| `overview.rs` `palette.rs` `switcher.rs` `navigation.rs` | Navigation surfaces and typed navigation actions |
| `history.rs` | Provider history rows, grouping, and adoption |
| `settings.rs` | Persisted-setting rows and their typed update commands |
| `usage.rs` | Ledger-derived usage rows. Never prices anything |
| `worktrees.rs` | The worktree panel: filters, dry run, and explicit confirmation |
| `attention.rs` | The pure attention reducer: triggers, replay suppression, unseen state |
| `notify.rs` | Bounded main-thread Notification Center bridge, click-to-run actions, fixed opt-in sound, and test fake |
| `menubar.rs` | Pure unseen-priority rollup plus the native macOS status item |
| `update.rs` | Compile-time feed pinning, bounded fetch and private staging, signed-bundle verification, and helper handoff |
| `query_editor.rs` `fuzzy.rs` `ansi.rs` `components.rs` | Text editing, ranking, terminal colour, shared elements |
| `preview.rs` | The deterministic cockpit fixture used by `examples/shell_preview` |

Two documents are normative for the shell:

- `docs/reference/COCKPIT.md` — the exact acceptance reference for the
  default window, with the image beside it.
- `docs/STATUS.md` — what is verified locally, what waits on a macOS
  permission, and what waits on signing and a hosted feed.

## Commands

```sh
cargo build --workspace   # build everything
cargo test --workspace    # run all tests (must pass before committing)
cargo run -p autoharnessd # run daemon
cargo run -p autoharness  # run desktop shell (Cmd-Q quits)

# The cockpit fixture: no daemon, no provider account, no client token.
cargo run -p autoharness-ui-gpui --example shell_preview
```

## Key invariants (from PLAN.md — do not break)

1. **The daemon persists every event to the store BEFORE broadcasting it.**
   See `AppState::emit` in `crates/daemon/src/lib.rs`.
2. Event sequences are monotonic; `events.subscribe` replays
   `seq > since_sequence` then streams live, without duplicates or gaps.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codejunkie99/autoharness](https://github.com/codejunkie99/autoharness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-08 -->
