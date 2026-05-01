---
trigger: always_on
description: `pgrust` is a PostgreSQL-style database prototype implemented in Rust. The repo is structured around the same broad layers PostgreSQL uses:
---

# pgrust — project guide for AI agents

## Overview

`pgrust` is a PostgreSQL-style database prototype implemented in Rust. The repo is structured around the same broad layers PostgreSQL uses:

- `src/backend/parser`: SQL grammar, parse tree handling, semantic analysis, and logical plan construction.
- `src/backend/executor`: expression evaluation, plan startup, runtime plan-node execution, tuple/value I/O, and aggregates.
- `src/backend/catalog`: table/type metadata and catalog mutations.
- `src/backend/access` and `src/backend/storage`: heap access, page layout, buffer/storage concerns.
- `src/backend/tcop` and `src/backend/libpq`: protocol entry points, error mapping, and frontend/backend message handling.
- `src/include/nodes`: shared node/value/plan/runtime data structures.
- `src/pgrust`: server/session/database orchestration outside the PostgreSQL-style backend tree.

The current codebase was recently refactored to separate parser, logical plan, and executor-runtime responsibilities more cleanly. Prefer extending those boundaries instead of reintroducing cross-layer dependencies.

## Worktrees

Run code changes inside a git worktree instead of the primary checkout. Multiple agents and humans frequently run against this repo in parallel on the same machine; sharing one working copy produces merge races, stale-index bugs, and half-applied edits across unrelated tasks.

Use a worktree when:

- The work will produce at least one commit or PR.
- The work spans more than one turn of file edits.

Skip the worktree for:

- Pure exploration, reading, or answering questions without file changes.
- One-off shell/inspection commands.

Conventions:

- Path: `../pgrust-worktrees/<short-descriptive-name>/` (sibling to this repo).
- Branch: follow the existing `<owner>/<short-description>` pattern where it applies (e.g. `malisper/alter-table-todo`, `jason/isolation-tests-setup`).
- Base: `perf-optimization`.
- Create: `git worktree add ../pgrust-worktrees/<name> -b <branch> perf-optimization`.
- Remove when merged or abandoned: `git worktree remove ../pgrust-worktrees/<name>`.

If you use Conductor (conductor.build), its workspaces live at `~/conductor/pgrust/<city>/` instead — no collision with the path above. Pick `perf-optimization` as the base in the New Workspace dialog, and rename the auto-generated city branch to match our `<owner>/<short-description>` pattern once you know what you're working on.

## Formatting

Rust formatting is pinned: `rust-toolchain.toml` fixes the rustc/rustfmt version and `rustfmt.toml` fixes the style edition. CI fails any PR that is not formatted with that exact rustfmt.

The repo ships a versioned pre-commit hook in `.githooks/pre-commit` that runs `cargo fmt -- --check`. Activate it with one command per clone (and per worktree, since worktrees have their own git config):

```sh
bash scripts/setup-dev.sh
```

This sets `core.hooksPath` to `.githooks` for that clone. After it runs, every `git commit` is rejected in <1 second if anything is unformatted. The setup is idempotent and safe to re-run.

- After editing any `*.rs` file, run `cargo fmt` before considering the task done. Running from inside the repo uses the pinned toolchain automatically.
- Do not reformat files you did not otherwise touch. If you notice unrelated drift, leave it — a separate fmt-only PR is the right cleanup path. Agents mixing stray reformatting into feature PRs is the exact churn this policy exists to prevent.
- Do not bypass the pre-commit hook with `git commit --no-verify`, and do not disable the `cargo-fmt-check` CI job. If the hook rejects you, run `cargo fmt`, re-stage, and re-commit.

### Optional: agent-side / editor-side auto-format

For an even tighter feedback loop, configure your tool to run `cargo fmt` immediately after every file edit so the pre-commit hook never even has to fire. These configs are per-user, not versioned in the repo:

- **Claude Code** — Add a `PostToolUse` hook in `~/.claude/settings.json`:

  ```json
  {
    "hooks": {
      "PostToolUse": [
        {
          "matcher": "Edit|Write|MultiEdit",
          "hooks": [{ "type": "command", "command": "cargo fmt" }]
        }
      ]
    }
  }
  ```

  Docs: https://code.claude.com/docs/en/hooks-guide

- **Codex CLI** — Hooks are still under development as of early 2026 (off by default). Enable with `codex_hooks = true` in the `[features]` section of your Codex config, then add a `PostToolUse` hook in `hooks.json` next to your config. Docs: https://developers.openai.com/codex/hooks

- **Cursor** — Enable VS Code's `editor.formatOnSave` setting and rust-analyzer will run rustfmt on every save. Settings → search "Format On Save" → check the box.

- **Conductor** (parallel agent runner) — Uses Claude Code under the hood, so the Claude Code `PostToolUse` hook above applies automatically.

## PostgreSQL Reference Repo

A sibling PostgreSQL source checkout is available at `../postgres`.

Guidance:

- Refer to the PostgreSQL repo as much as necessary when implementing SQL behavior, planner/executor semantics, catalog behavior, protocol details, or PostgreSQL-compatible errors.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [malisper/pgrust](https://github.com/malisper/pgrust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
