---
trigger: always_on
description: This file provides guidance to agents working in this repository.
---

# AGENTS.md

This file provides guidance to agents working in this repository.

## Build & Test

```bash
cargo build                    # compile
cargo test                     # run the full test suite
cargo test state::tests        # run tests in a specific module
cargo test resolve_selection   # run tests matching a name pattern
```

## CI

`.github/workflows/ci.yml` runs on every PR and every push to `main`. Five jobs, all required to pass:

```bash
cargo test   --all-targets --locked                   # Test
cargo fmt    --all --check                            # Rustfmt — formatting
cargo clippy --all-targets --locked -- -D warnings    # Clippy — lints as errors
cargo audit                                           # Security audit — RustSec advisories
bash scripts/verify-agent-hooks.sh                    # Agent hook protocol — see below
```

Run `cargo fmt --all` and `cargo clippy --all-targets -- -D warnings` before pushing. CI pins the toolchain to **Rust 1.96.1** (see `RUST_VERSION` in `ci.yml`); rustfmt reflows and clippy lints shift between releases, so format/lint with a matching toolchain to avoid CI turning red on formatting alone.

## Workflow

Every change — even a tiny one — happens in a **fresh git worktree on a new branch**, then goes out as a PR. Never edit and commit directly on `main` in the primary checkout.

```bash
git worktree add ../tws-<slug> -b <branch> origin/main
cd ../tws-<slug>
# ...edit, commit, push, gh pr create
```

This keeps in-flight work isolated from `main`, and ensures every change is reviewable on GitHub before it lands.

## Release

Patch/minor/major bumps follow semver.

1. Land fix/feature commits on `main` (merge any feature worktrees in first — do **not** bump versions inside a feature branch/worktree, it causes `Cargo.lock` conflicts on merge).
2. In the primary `main` checkout, bump `version` in both `Cargo.toml` and `Cargo.lock` (the `[[package]] name = "tws"` entry) in a separate commit titled `version bump to vX.Y.Z`.
3. Lightweight tag at the bump commit: `git tag vX.Y.Z`. Push both: `git push origin main && git push origin vX.Y.Z`.

Don't use `--follow-tags` (only pushes annotated tags) or `--tags` (pushes all local tags, including any forgotten experimental ones).

## What This Is

tws is a standalone Rust TUI that replaces tmux's `prefix+s` session picker. It adds a persistent organizational hierarchy on top of ephemeral tmux sessions:

```
Collection → Thread → Session(s)
```

Collections and threads are user-created, persisted to `~/.config/tws/state.json`. Sessions are live tmux sessions discovered at runtime. Agent sessions (Claude Code, Codex) are detected by scanning tmux panes and matching child process names.

## Architecture

**Single-threaded event loop** in `app.rs` — the brain of the app. It owns the `Mode` state machine, key routing, rendering, and all side effects. The loop polls keys every 250ms and refreshes tmux sessions on a 30s floor, plus immediately whenever an agent hook fires (see [Agent status protocol](#agent-status-protocol)).

### Mode state machine

```
Mode::Normal → Mode::Input { purpose, buffer } → confirm → back to Normal
             → Mode::Confirm { purpose }       → confirm → back to Normal
             → Mode::Finder { ... }            → select  → back to Normal
```

`InputPurpose` and `ConfirmPurpose` enums capture *what* the modal is for (add collection, rename thread, kill session, etc.) at open time. On confirm, the purpose is consumed via `std::mem::replace` to avoid borrow conflicts on `self.mode`.

### Selection resolution

The tree widget (`tui-tree-widget`) uses UUID strings as node identifiers. `state.rs::resolve_selection()` maps a `&[String]` tree path into `SelectedItem` — an enum with variants `None | Collection(idx) | Thread(col, thread) | Session(col, thread, sess) | Agent(col, thread, sess, agent)`. This is the bridge between the tree widget and the domain model.

### Key modules

| Module | Role |
|---|---|
| `app.rs` | Main loop, mode state machine, key routing, rendering |
| `core/model.rs` | Data structs: Collection, Thread, Session, AgentSession, AgentType |
| `core/state.rs` | AppState, CRUD methods, `resolve_selection()`, session/agent lookups |
| `core/persistence.rs` | JSON save/load to `~/.config/tws/` (state + UI state) |
| `core/notes.rs` | File-based notes stored as `.md` in `~/.config/tws/notes/` |
| `tmux/commands.rs` | Thin wrappers around `tmux` CLI subcommands via `std::process::Command` |
| `tmux/agent_scan.rs` | Detect AI agents by `tmux list-panes` + `ps -e`, match child process names |
| `components/` | Stateless render functions: tree_view, input_modal, confirm_modal, finder_modal, notes_sidebar, status_bar, recent_bar |
| `theme.rs` | All `Style` constants — warm palette (orange collections, tan threads, sage green sessions) |

### Rendering

Immediate-mode: all widgets are rebuilt from `AppState` each frame. Components are stateless functions (`fn render(frame, state, area)`), not structs. `ratatui` diffs the output for efficiency.

### tmux integration

- Sessions are launched detached (`tmux new-session -d`), then attached via `switch-client` (inside tmux) or `attach-session` (outside tmux)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ytaskiran/tws](https://github.com/ytaskiran/tws) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
