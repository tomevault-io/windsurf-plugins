---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is arbor?

A CLI tool for managing git worktrees, written in Rust. It organizes worktrees under `~/.arbor/worktrees/` and bare repos under `~/.arbor/repos/`, configured via `~/.arbor/config.toml`.

## Build and test commands

```sh
cargo build                    # Build debug
cargo build --release          # Build release
cargo install --path .         # Install to ~/.cargo/bin/
cargo test                     # Run all tests (unit + integration)
cargo test --lib               # Unit tests only
cargo test --test add          # Run a single integration test file
cargo test <test_name>         # Run a single test by name
cargo fmt                           # Format code
cargo clippy --tests -- -D warnings # Lint (warnings treated as errors)
```

## Pre-commit checks

**Always run `cargo fmt` and `cargo clippy --tests -- -D warnings` after making changes, before committing.** Fix any issues they report before creating a commit.

## Architecture

- **`src/main.rs`** — Entry point. `main()` resets SIGPIPE to default (so piping stdout doesn't panic), parses CLI args, configures color, calls `run()`, and formats errors with `display::print_error`. `run()` dispatches to command handlers.
- **`src/cli.rs`** — CLI definition using clap derive. Defines `Cli` struct and `Command` enum (Add, Switch, List, Remove, Dir, Clone, Clean, Prune, Fetch, Init). Add has `--base` / `-b` to start a new branch from a specific ref. Switch branch is optional (interactive fuzzy selection when omitted). Remove branch is optional (interactive fuzzy selection when omitted, `"."` removes current worktree). Init has an `--inject` flag for non-interactive shell config injection.
- **`src/config.rs`** — Loads/creates `~/.arbor/config.toml` with tilde expansion. Uses serde + toml.
- **`src/git/`** — All git operations via `std::process::Command`, split into submodules:
  - `runner.rs` — `run_git`, `run_git_output`, `run_git_inherited` (`pub(super)`)
  - `types.rs` — `ParsedWorktree`, `WorktreeInfo`, `Tracking`, `parse_worktree_list`, `sanitize_branch`, `strip_git_suffix` + unit tests
  - `commands.rs` — All pub fn git wrappers (`repo_toplevel`, `show_file_from_head`, `worktree_infos`, `resolve_worktree_branch`, `delete_branch`, etc.)
  - `mod.rs` — Re-exports all pub items (callers use `crate::git::*` unchanged)
- **`src/hooks.rs`** — Post-create hook support and per-project worktree directory overrides. `ProjectConfig` has an optional `worktree_dir` field. Key public functions: `resolve_worktree_dir` (resolves absolute/tilde/relative paths against a repo root), `load_worktree_dir_from_path` (reads `.arbor.toml` from filesystem), `load_worktree_dir_from_git` (reads `.arbor.toml` via `git show HEAD:`). Executes `post_create` commands via shell with stdout redirected to stderr.
- **`src/display.rs`** — Colored terminal output (using `colored` crate — only file that imports it). Color configuration (`configure_color`), interactive worktree selection (`fuzzy_select_worktree`), cwd-escape helper (`escape_dir_if_cwd_inside`), table formatting for worktree listings, summary stats, path shortening (`shorten_path`), terminal-aware path output (`print_path_hint`), and user-facing messages (`print_ok` ✓, `print_error` ✗, `print_note` ▸, `print_section`, `print_heading`, `print_hint`, `print_cd_hint`).
- **`src/commands/`** — One file per subcommand (add, clean, clone, dir, fetch, init, list, prune, remove, switch). Each exports a `run` function re-exported from `commands/mod.rs`. `remove` finds worktrees via `git::resolve_worktree_branch` (no dependency on `Config`).

## Key conventions

- Commands print the worktree path to **stdout** (for `cd $(arbor add ...)` workflows) and user messages to **stderr** via the `display` module. Commands that remove the user's cwd (`remove`, `clean`) also print a path to stdout so the shell wrapper can cd out of the deleted directory.
- All user-facing messages (errors, notes, success) start with a capital letter.
- All colored output goes through `src/display.rs` — no other file imports `colored`.
- Interactive prompts use `dialoguer` (`arbor clean` uses multi-select, `arbor switch`/`dir`/`remove` use `display::fuzzy_select_worktree`).
- Branch slashes become dashes in directory names (e.g., `feature/auth` → `feature-auth`).
- Error handling uses `anyhow::Result` throughout.
- Only add comments where the logic isn't self-evident. Do not add comments that restate what the code does.
- Rust edition 2024.
- Commit messages follow [conventional commits](https://www.conventionalcommits.org/): `feat:`, `fix:`, `chore:`, `ci:`, `docs:`, `refactor:`, `test:`. Use `feat!:` or a `BREAKING CHANGE:` footer for breaking changes. Only `feat:` and `fix:` trigger version bumps — use `ci:` for CI/workflow changes and `chore:` for other non-user-facing changes.

## Releasing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [morellodev/arbor](https://github.com/morellodev/arbor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
