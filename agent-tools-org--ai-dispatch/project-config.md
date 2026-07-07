---
trigger: always_on
description: - NEVER cp binary to `/opt/homebrew/bin/` — macOS provenance xattr blocks execution
---

# ai-dispatch (aid)

## Install

- NEVER cp binary to `/opt/homebrew/bin/` — macOS provenance xattr blocks execution
- `/opt/homebrew/bin/aid` is a symlink to `~/.cargo/bin/aid`
- Install command (MUST re-sign after copy — sandbox provenance blocks execution):
  ```bash
  cp "$CARGO_TARGET_DIR/release/aid" ~/.cargo/bin/aid && codesign --force --sign - ~/.cargo/bin/aid
  ```

## Release

Release must go through `scripts/release.sh`. Do not manually bump `Cargo.toml`, edit the top release entry in `CHANGELOG.md`, create the release commit, create the release tag, or push the release branch/tag by hand.

Release flow requirements:
- Start from a clean git worktree. Commit or stash local edits before running the release script.
- Prepare a Markdown notes file with one `- ` bullet per shipped change.
- Run `scripts/release.sh --dry-run <version> <notes-file>` first and review the planned commit/tag/push.
- Run `scripts/release.sh <version> <notes-file>` for the actual release.
- Treat any direct `git tag`, manual version bump, or manual changelog-only release edit as an invalid release flow.

```bash
cat > /tmp/aid-release-notes.md <<'EOF'
- Short release summary
- Additional shipped change
EOF

scripts/release.sh --dry-run 8.75.0 /tmp/aid-release-notes.md
scripts/release.sh 8.75.0 /tmp/aid-release-notes.md
```

## Run

Dispatch a task to an AI agent. Core command — most other features build on this.

```bash
aid run codex "Add unit tests" --verify              # with auto-verify
aid run gemini "Research topic" -o notes.md           # research with output file
aid run codex "Refactor" -w feat/refactor --bg        # background + worktree
aid run auto "implement feature" --team dev           # auto-select agent with team context
```

Built-in run agents: `gemini`, `agy`, `codex`, `copilot`, `opencode`, `cursor`, `kilo`, `mimocode`, `codebuff`, `droid`, `oz`, `claude`, and `auto`.

`agy` (Antigravity CLI) is a drop-in successor to `gemini` for Google One / Gemini Code Assist (individuals) users. After June 18, 2026, gemini stops serving those tiers — switch to `agy`. Paying API users keep using `gemini`.

### Key flags

| Flag | Purpose |
|------|---------|
| `-w, --worktree <branch>` | Run in isolated git worktree |
| `--verify [<cmd>]` | Auto-verify on completion (default: project verify cmd) |
| `--judge [<agent>]` | AI judge evaluates output quality |
| `--peer-review <agent>` | Dispatch peer review after completion |
| `--cascade <agents>` | Comma-separated fallback agents on failure |
| `--context <file>...` | Inject files as context into the prompt |
| `--context-from <task-id>...` | Inject output from previous tasks as context |
| `--scope <path>...` | Restrict agent file access to specific paths |
| `--skill <name>...` | Inject methodology skills into the prompt |
| `--template <name>` | Wrap prompt with a template |
| `--on-done <cmd>` | Shell command to run on task completion |
| `--hook <spec>...` | Hook specs for the dispatched task |
| `--bg` | Run in background (non-blocking) |
| `--sandbox` | Run agent in sandboxed mode |
| `--container <image>` | Run agent inside a container |
| `--best-of <N>` | Run N copies, pick best result |
| `--metric <cmd>` | Custom metric command for best-of selection |
| `--budget` | Use budget-optimized model |
| `--read-only` | Agent cannot modify files |
| `--idle-timeout <secs>` | Kill agent if idle for N seconds |
| `--retry <N>` | Auto-retry on failure (default: 0) |
| `-g, --group <wg-id>` | Assign to workgroup |

## Batch

Dispatch multiple tasks from a TOML file.

```bash
aid batch tasks.toml --parallel                     # parallel dispatch
aid batch tasks.toml --parallel --max-concurrent 3  # limit concurrency
aid batch tasks.toml --analyze                      # warn about file overlaps
aid batch tasks.toml --wait                         # block until all complete
aid batch tasks.toml --var key=value                # template variables
aid batch init                                      # generate template TOML
aid batch retry --group wg-abc1                     # re-dispatch failed tasks
```

### Batch TOML format

```toml
[defaults]
dir = "."
agent = "codex"
team = "dev"
verify = "cargo check"
fallback = "cursor"
model = "o3"
context = ["src/types.rs"]
skills = ["implementer"]
worktree_prefix = "feat/my-feature"    # auto-generates worktree per task
analyze = true                          # warn about overlapping file edits
max_duration_mins = 30                  # hard timeout in minutes for batch tasks

[[task]]
name = "parser"                         # REQUIRED if sharing worktree with other tasks
prompt = "Implement parser"
worktree = "feat/my-feature/parser"
depends_on = ["types"]                  # wait for named task to complete
fallback = "oz,opencode"
on_success = "tests"                    # trigger conditional task on success
on_fail = "cleanup"                     # trigger conditional task on failure
conditional = true                      # only runs when triggered by on_success/on_fail
idle_timeout = 120
```

- `context`, `skills`, `scope` accept both string and array: `context = "file.md"` or `context = ["a.md", "b.md"]`
- `fallback` supports comma-separated agents: `fallback = "oz,opencode,codex"`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agent-tools-org/ai-dispatch](https://github.com/agent-tools-org/ai-dispatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
