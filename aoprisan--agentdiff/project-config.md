---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`agentdiff` is a Rust TUI git-diff tool built for one workflow: a developer runs Claude Code in auto/accept-edits mode, then reviews the resulting working-tree changes. The review is the only human control point, so the tool optimizes for fast triage, **agent-intent correlation** (showing what the agent *said* it was doing, pulled from Claude Code's own session data), and a personal review checklist. It is **read-only** — it never mutates the working tree.

## Build / test / run

```bash
cargo build                       # compile
cargo clippy --all-targets        # lint — keep this clean (CI bar)
cargo test                        # all tests
cargo test <name>                 # single test, e.g. cargo test diff_round_trips_through_serde
cargo run                         # launch the TUI (needs a real terminal/TTY)
cargo run -- --help               # CLI surface (non-interactive smoke check)
```

Snapshot tests use [`insta`]. When a snapshot is new or intentionally changed, accept it with `INSTA_UPDATE=always cargo test` (or `cargo insta review` if `cargo-insta` is installed). Pending `*.snap.new` files are gitignored — never commit them; commit the accepted `.snap`.

## Phased delivery — read this before starting work

The implementation is delivered **one phase at a time**. The plan is the source of truth for *what* to build and in what order:

- Start at [`docs/plan/README.md`](docs/plan/README.md) (index + status), then [`docs/plan/00-overview.md`](docs/plan/00-overview.md) for the full architecture, stack, domain types, and risks.
- Each `docs/plan/0N-phaseX-*.md` is self-contained: goal, in/out of scope, crates to add, ordered tasks, files touched, acceptance criteria. **Do not pull scope forward from a later phase.**
- Phases 0–3 and 6 are **complete** (worktree reviewer, Claude Code per-run scoping + intent correlation, live re-diff, session picker, config, verification surfacing — plus a GitHub Copilot CLI provider, `--report`/`--report=json`, search, and editor jump-out beyond the original plan). Phase 4 (risk engine) is optional and next; Phase 5 (action layer) stays out of scope by the read-only decision.
- These docs are mirrored from `~/.claude/plans/agentdiff/`. If you revise a phase mid-build, update the `docs/plan/` copy so the repo stays the source of truth.

## Architecture (the big picture)

**The `Diff` is the spine.** `git`/session producers build a `domain::Diff`; risk, intent, and review state all anchor onto the content-addressed `HunkRef` inside each hunk (a fingerprint, *not* a line number) so they re-attach across a live re-diff even as the tree changes underneath. The "before" source (`DiffBase`) only changes how a `Diff` is *built* — everything downstream is identical. The default base is the latest Claude Code **agent run** (diffing its pre-run file-history backups vs the working tree); it falls back to working-tree-vs-HEAD when no session is found.

**Module dependency direction is strict and acyclic:**
- `domain/` — pure types + pure transforms, **no I/O**. The contract every other module anchors to. `domain::ids` is the **persisted** fingerprint hash (hand-rolled FNV-1a with a pinned-value test) — never swap in `DefaultHasher` or any unstable hasher.
- `git/`, `session/`, `watch/` — depend only on `domain`. The only place libgit2 / filesystem / agent-session parsing lives. `session/` hosts one provider per submodule (Claude Code at the root, `copilot/`), both producing the same `SessionContext`. All recorded-path ↔ repo matching goes through `session::paths::RepoPaths` (symlink/`..`-tolerant) — never a raw `strip_prefix`.
- `app/` — UI-framework-agnostic core: `AppState` + the single `(state, event)` reducer in `update.rs`. Orchestrates I/O modules via channels. No ratatui types leak out beyond the input `Event`.
- `tui/` — ratatui rendering only; reads `AppState`, contains no business logic.

**Concurrency model: no async runtime.** Threads + `crossbeam-channel`. An input thread does blocking reads; the main thread owns `AppState` + the terminal and renders. Long ops (git diff, transcript parse, syntect) belong on a worker thread, and a **generation counter** drops results superseded by a newer re-diff. Don't reach for tokio.

**Rendering invariants (hold from Phase 1 on):** the diff pane is **virtualized** (flatten the `Diff` to a row index once; render only the visible window) and syntax highlighting via `syntect` is **lazy** (visible rows + small overscan, LRU-cached). Never fully highlight a large file. Word-diff spans are computed at model-build time and stored on each `Line`.

### Claude Code session integration (the differentiator, Phase 2)

These data sources were verified to exist on disk; see `docs/plan/00-overview.md` and `03-phase2-cc-integration.md` for specifics:
- Transcripts: `~/.claude/projects/<slug>/<session-uuid>.jsonl` (`slug` = absolute cwd with `/` and `.` → `-`).
- Pre-edit file content: `~/.claude/file-history/<session-uuid>/<backupFileName>`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aoprisan/agentdiff](https://github.com/aoprisan/agentdiff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
