---
trigger: always_on
description: All code changes go through a branch, worktree, and PR — no exceptions.
---

# AGENTS.md

## Development workflow

All code changes go through a branch, worktree, and PR — no exceptions.

- **Branch + worktree + PR for every change.** Create a fresh branch off latest `main`, materialize it as a git worktree under `.claude/worktrees/<branch-name>`, make changes there, and open a PR. The top-level checkout at `~/agentd` stays on `main` — never edit files there directly.
- **No direct push to `main`.** Changes land on `main` only via a merged PR.
- **Release process lives in [`docs/RELEASING.md`](docs/RELEASING.md).** Use that guide for versioned releases and publishing prebuilt binaries.
- **No `Co-Authored-By: Claude` trailer in commits.** Don't append model attribution to commit messages. `Co-authored-by:` for other humans is fine.
- **Clean up after merge.** Remove the worktree (`git worktree remove <path>`), delete the local branch (`git branch -d <name>`), and delete the remote branch (e.g. via GitHub's "delete branch after merge", or `git push <remote> --delete <name>`).
- **After merge, update and build the main worktree.** Once a PR is merged and the feature worktree is cleaned up, switch to the top-level checkout (`~/agentd`), pull latest `main`, and run `cargo build` there (debug profile). This keeps the user's main worktree binaries current so `/construct restart` can pick up the latest merged `construct` changes, especially when operating from a remote-control session. Report the updated main-worktree debug binary paths when relevant.
- **When the change is testable, build all binaries in the worktree and report paths in the agent response.** Run `cargo build` inside the worktree (debug profile — much faster to iterate on than release; the binaries live under `.claude/worktrees/<branch>/target/debug/`), then print the absolute path of every binary the workspace produces — `construct` binary the workspace produces in the agent response so the user can copy and run it. Explicitly call out *which* binary the PR's code lives in so the user can run the right one without grepping the diff (e.g. "this PR only touches `crates/cli` → relevant binary is `construct`").
- **Record a video / screenshot when it helps the reviewer, and post accessible artifacts on the PR.** This is a judgment call:
  - Sometimes only an "after" recording makes sense (a brand-new pane / popup / view that didn't exist before).
  - Sometimes a before/after pair is needed (a tweak to an existing render: a color, a fade rate, a layout shift).
  - Sometimes neither is useful (refactors, internal API changes, daemon logic with no user-visible surface).
  Use [Recording the TUI](#recording-the-tui) below for the mechanics. Report local artifact paths in the agent response so the user can open them; if posting on the PR, attach or upload the actual media so reviewers can access it. **When unsure which of the three applies, ask the user before recording.**

## Recording the TUI

Use [vhs](https://github.com/charmbracelet/vhs) to capture deterministic mp4 / gif clips of the TUI without needing a desktop session or screen-recording permissions. The notes below are the ones we wish we'd had on the first attempt.

- **Build the worktree's binaries.** vhs records whatever `construct` you point it at, so make sure the worktree has been built (`cargo build` per the workflow above) before recording. For a before/after pair, prepare two worktrees so each side has its own binaries — never re-record `before` from a tree that already has the change applied.
- **Isolated daemon.** Run vhs against a fresh `CONSTRUCT_RUNTIME_DIR` / `CONSTRUCT_STATE_DIR` / `CONSTRUCT_DATA_DIR` / `CONSTRUCT_CONFIG_DIR` under `/tmp/` so it doesn't collide with the user's running daemon. Each recording gets its own dir and its own daemon process; tear them down at the end.
- **Put the TUI in a state that actually shows your change.** This part varies most by change — pick whichever shape fits:
  - **Specific harness features** (a smith tool, codex output rendering, claude resume, …): spawn that harness with a representative prompt, e.g. `construct new --prompt "<task>" smith`. Use a prompt whose output exercises the diff (tool calls if you changed tool rendering, long messages if you changed wrapping, etc.).
  - **Minibuffer / keymap / popup / palette**: send the keystrokes from inside the vhs tape with `Type`, `Ctrl+X`, `Enter`, `Sleep`, etc. — no extra sessions needed if the feature is reachable from a stock TUI.
  - **Session-list / modeline / matrix rain / anything driven by fleet activity**: spawn 2–4 sessions producing ambient activity. The most robust pattern is `construct new shell` (interactive shell) followed by `construct send <id> "<command>"` pushing a noise loop into each. *Don't* pass the loop as the `new shell` prompt — both bash and zsh observed to fall back to interactive mode under PTY and never actually run `-lc <cmd>`, leaving the daemon silent.
  - **Single-session views** (transcript, scrollback, diff): spawn one session, then trigger the view via tape keystrokes (`C-x z` for zoom, mouse-wheel events, etc.).
  Whatever the shape, give the daemon a few seconds to settle (`sleep 3`) after setup so the first frames the tape captures aren't a half-loaded UI.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [construct-worlds/construct](https://github.com/construct-worlds/construct) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-02 -->
