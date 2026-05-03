---
trigger: always_on
description: cargo run -- hook pre-merge --yes   # run all tests + lints (do this before committing)
---

# Worktrunk Development Guidelines

## Quick Start

```bash
cargo run -- hook pre-merge --yes   # run all tests + lints (do this before committing)
```

For Claude Code web environments, run `task setup-web` first. See [Testing](#testing) for more commands.

## Project Status

**This project has a growing user base. Balance clean design with reasonable compatibility.**

We are in **maturing** mode:
- Breaking changes to external interfaces require justification (significant improvement, not just cleanup)
- Prefer deprecation warnings over silent breaks
- No Rust library compatibility concerns (this is a CLI tool only)
- **MSRV policy: latest stable − 1** — bumped during weekly tend maintenance (see `running-tend` skill)

**External interfaces to protect:**
- **Config file format** (`wt.toml`, user config) — avoid breaking changes; provide migration guidance when necessary
- **CLI flags and arguments** — use deprecation warnings

Everything else (internal APIs, output formatting, log locations) is flexible. Prefer the best technical solution; use deprecation warnings when external interfaces must change.

## Terminology

Use consistent terminology in documentation, help text, and code comments:

- **main worktree** — the original git directory (from clone/init); bare repos have none
- **linked worktree** — worktree created via `git worktree add` (git's official term)
- **primary worktree** — the "home" worktree: main worktree for normal repos, default branch worktree for bare repos
- **default branch** — the branch (main, master, etc.), not "main branch"
- **target** — the destination for merge/rebase/push (e.g., "merge target"). Don't use "target" to mean worktrees — say "worktree" or "worktrees"

## Skills

Check `.claude/skills/` for available skills and load those relevant to your task.

Key skills:

- **`writing-user-outputs`** — Required when modifying user-facing messages, hints, warnings, errors, or any terminal output formatting. Documents ANSI color nesting rules, message patterns, and output system architecture.

## Testing

See `tests/CLAUDE.md` for test infrastructure, assertion style, and test granularity guidelines.

### Running Tests

```bash
# All tests + lints (recommended before committing)
cargo run -- hook pre-merge --yes

# Tests with coverage report → target/llvm-cov/html/index.html
task coverage
```

**For faster iteration:**

```bash
pre-commit run --all-files              # lints only
cargo test --lib --bins                 # unit tests only
cargo test --test integration           # integration tests (no shell tests)
cargo test --test integration --features shell-integration-tests  # with shell tests
```

### Claude Code Web Environment

Run `task setup-web` to install required shells (zsh, fish, nushell), `gh`, and other dev tools. Install `task` first if needed:

```bash
sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b ~/bin
export PATH="$HOME/bin:$PATH"
task setup-web
```

The permission tests (`test_permission_error_prevents_save`, `test_approval_prompt_permission_error`) skip automatically when running as root.

### Shell/PTY Integration Tests

PTY-based tests (approval prompts, TUI picker, progressive rendering, shell wrappers) are behind the `shell-integration-tests` feature.

**IMPORTANT:** Tests that spawn interactive shells (`zsh -ic`, `bash -ic`) cause nextest's InputHandler to receive SIGTTOU when restoring terminal settings. This suspends the test process mid-run with `zsh: suspended (tty output)` or similar. See [nextest#2878](https://github.com/nextest-rs/nextest/issues/2878) for details.

**Solutions:**

1. Use `cargo test` instead of `cargo nextest run` (no input handler issues):
   ```bash
   cargo test --test integration --features shell-integration-tests
   ```

2. Or set `NEXTEST_NO_INPUT_HANDLER=1`:
   ```bash
   NEXTEST_NO_INPUT_HANDLER=1 cargo nextest run --features shell-integration-tests
   ```

The pre-merge hook (`wt hook pre-merge --yes`) already sets `NEXTEST_NO_INPUT_HANDLER=1` automatically.

## Documentation

**Behavior changes require documentation updates.**

When changing:
- Detection logic
- CLI flags or their defaults
- Error conditions or messages
- Config fields or sections — also update `dev/*.example.toml` (these are embedded in CLI help via `include_str!`, so stale examples propagate to docs and snapshots)

Ask: "Does `--help` still describe what the code does?" If not, update `src/cli/mod.rs` first.

### Auto-generated docs

Documentation has three categories:

1. **Command pages** (config, hook, list, merge, remove, step, switch):
   ```
   dev/*.example.toml (included via include_str!)
       ↓
   src/cli/mod.rs (PRIMARY SOURCE)
       ↓ test_docs_are_in_sync
   docs/content/{command}.md → skills/worktrunk/reference/{command}.md
   ```
   Edit `src/cli/mod.rs` (`after_long_help` attributes), never the docs directly.

2. **Non-command docs** (claude-code, faq, llm-commits, tips-patterns, worktrunk):
   ```
   docs/content/*.md (PRIMARY SOURCE)
       ↓ test_docs_are_in_sync
   skills/worktrunk/reference/*.md
   ```
   Edit the docs file directly. Skill reference is auto-synced.

3. **Skill-only files** (shell-integration.md, troubleshooting.md):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [max-sixty/worktrunk](https://github.com/max-sixty/worktrunk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
