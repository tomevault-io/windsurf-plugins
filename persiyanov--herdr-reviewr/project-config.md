---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. AGENTS.md is the primary file. CLAUDE.md is a symlink to it.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. AGENTS.md is the primary file. CLAUDE.md is a symlink to it.

herdr-reviewr is a Rust TUI (ratatui) code-review pane: it runs in a [herdr](https://herdr.dev) pane beside a coding agent, shows the agent's diff, takes line comments, and sends them back to the agent's input. One binary, one git worktree per pane. It also runs standalone (`cargo run` in any repo).

## Commands

- `just test` — full test suite. Single test: `cargo test <name>` (unit tests live beside the code, integration tests in `tests/`: `cargo test --test app_flow <name>`).
- `just lint` — clippy with warnings as errors. `just fmt` / `just fmt-check` — rustfmt.
- `just ci` — exactly what CI runs (fmt-check, lint, test, release build).
- `just qa-install` — put a local build into the user's real herdr panes. See "QA install" below before using it.
- `just smoke-edit` — PTY smoke test of the editor path (`e`) against a real release binary. Unit tests stop at the argv; everything after it is terminal state, so run this after any change to `run_editor`, the terminal mode stack, or the editor dialects. Not part of `just ci`: it drives a pty and takes about a minute.
- `python3 scripts/bench_tui.py --binary target/release/herdr-reviewr --fixture` — perceived-latency benchmark (keypress → painted frame, via PTY), the acceptance instrument. `cargo run --release --example bench_latency -- <repo>` attributes a slow number to its component calls. The one committed baseline is `scripts/bench-results/baseline.json` — replace it when a change moves the numbers, never add per-round runs. Run before/after any change to the reload, render, git, or highlight paths, and compare medians A/B under the same system load (rebuild the old binary to a second target dir and interleave runs — absolute numbers drift with background load).

## Invariants

New behavior is designed with `/brainstorming` and sequenced with `/planning` in the conversation; the repo keeps no spec tree. The commit message and the changelog carry the decisions.

Load-bearing invariants. Cite them by name:

- **No writes**: reviewr never mutates the worktree, index, or branches. Its only git writes are private refs under `refs/worktree/reviewr/`: the turn baseline and the base pick.
- **Comments survive**: comments are never lost to a refresh or the agent's edits, and leave only by explicit export. The comment store is in-memory **by design** — do not propose persisting it.
- **Continuity**: place state (cursor, scroll, tab, scope, folds, selection, layout) moves only under the user's own input. World events (polls, refreshes, fetch results) may only *reconcile* it: match by identity first (path, comment author+anchor — never row index), fall back to the nearest surviving target, clamp last. Derived state on screen may be stale, never wrong: blank a view only when its identity changed, never because the same thing gained newer content.

## Architecture

The runtime is a single-threaded frame loop (`event_loop` in `src/lib.rs`): draw → wait for input or poll deadline → mutate `App` → draw. Clipboard, agent-send, and per-file diff builds run synchronously between frames, and a terminal editor holds the loop for its whole session by design (`policies/ux-responsiveness.md`). Five things run on worker threads: the world worker (`src/world.rs` — the refresh build and turn tracking), the search worker (`src/search.rs` — the fff-search engine, which runs its own scan, watch, and content-index threads), the PR input probe, the PR forge fetch (`gh`/`glab`/`az`), and config recovery. World results land through `land_world_completion`: input-tagged, latest-wins, reconciled only while the view still matches (see Continuity above).

- `src/app.rs` — the `App` state machine. Tabs (`Changes`/`AllFiles`/`Pr`), scopes (`Uncommitted`/`Branch`/`LastTurn`), `Focus` (files vs diff pane), `Mode` (`Normal`, the `Composing`/`List` overlays, and the body-replacing `Search` screen). `reconcile_world()` is the one place a world snapshot touches place state; `reload()` is the synchronous build+reconcile pair used at startup, first tab visits, and scope switches. Each file tab stashes its full place state on switch-away (`swap_active_with_stash`). While composing, the open diff is frozen (reconcile skips it) so a draft's anchor can't move.
- `src/world.rs` — the world worker: the pure snapshot build (`WorldInput` → `WorldSnapshot`), the request/completion channels (latest-wins by generation), and `TurnHost` (the turn tracker, worktree snapshots, and the baseline ref write, all worker-side).
- `src/git.rs` — every git subprocess. `changed_files` (scope changesets), `all_files` (tracked + untracked + ignored via `ls-files` — never use `git status --ignored`, it walks inside ignored trees and costs seconds), `snapshot_worktree` (temp-index `add -A` + `write-tree` for turn baselines), baseline refs.
- `src/diff.rs` — `FileDiff` build (syntect highlight both sides, similar-line pairing, word emphasis, folds) and `DiffCache`, keyed by path and gated by content hash. Cleared on scope switch and theme change.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [persiyanov/herdr-reviewr](https://github.com/persiyanov/herdr-reviewr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
