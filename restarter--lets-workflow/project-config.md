---
trigger: always_on
description: Claude Code plugin for development workflow with session management, code review, and task tracking.
---

# lets-workflow

Claude Code plugin for development workflow with session management, code review, and task tracking.

## Structure

Monorepo layout: plugin source in `plugins/lets/` subdirectory, marketplace manifest at root pointing into it. Infrastructure scripts and docs stay at root, outside the plugin payload.

```
.claude-plugin/marketplace.json   # Marketplace manifest (source: ./plugins/lets)
plugins/lets/                     # Plugin payload (everything ${CLAUDE_PLUGIN_ROOT} resolves to)
├── .claude-plugin/plugin.json    #   Plugin manifest (Claude Code; .codex-plugin/ planned for multi-agent)
├── commands/                     #   Slash commands (/lets:start, /lets:done, /lets:review, etc.)
├── agents/                       #   15 expert agents (review/opinion/ask/plan/backlog/team; skeptic verifies review findings + research claims)
├── skills/                       #   Reusable skills (user-facing auto-triggered + internal referenced by commands)
├── rules/lets-rules.md           #   Workflow rules (frontmatter `version`; copied to .claude/rules/ by `lets init`)
└── hooks/                        #   SessionStart + PreCompact hooks (drift check + LETS Config only - no rules in stdout)
cli/                              # Go CLI - companion binary (Phase 2+, lets-7vtaw)
├── cmd/lets/main.go              #   Entry point (thin)
├── internal/cli/                 #   Cobra command factories (root.go, version.go, *_test.go)
├── internal/version/version.go   #   Version (var, ldflags-overridable from git tag)
├── go.mod, go.sum
└── .golangci.yml                 #   golangci-lint v2 config (default linters + misspell; gofmt/goimports as formatters)
Makefile                          # Repo-root build (build/test/vet/lint/fmt/install/clean)
.editorconfig                     # Editor whitespace/charset settings
.github/workflows/                # CI: ci.yml (Go build/vet/test/lint on PRs+pushes), verify-versions.yml (source-tree version coherence), release.yml (tag-driven goreleaser)
scripts/dev/                      # Dev workflow helpers (make dev / dev-tmux)
scripts/release/                  # Release tooling: bump-version.sh + verify-versions.sh (used by Makefile bump/release-tag)
scripts/remote/dolt/              # Dolt SQL server VPS deployment + ad-hoc backup (NOT plugin)
scripts/remote/beads-web/         # beads-web (Rust kanban board) VPS deployment (NOT plugin)
scripts/deprecated/               # Retired scripts kept for cleanup runbooks - gitignored, not tracked
docs/                             # Public-facing docs (installation.md, commands.md, workflow.md, agents.md, autonomous.md, commands/ per-command pages, …) + images/; planning notes / comment exports / KB live in gitignored docs-local/
reference/                        # Local-only reference materials (gitignored)
```

References that resolve via `${CLAUDE_PLUGIN_ROOT}` (e.g. `${CLAUDE_PLUGIN_ROOT}/skills/X/SKILL.md`) work as before - the env var points at `plugins/lets/`.

## Local Development

Testing the LETS plugin against unmerged changes — across parallel worktrees, in TMUX, without polluting the global installation:

- **`make dev`** in a worktree builds `cli/lets` with version `dev-<branch>-<sha>[-dirty]`, prepends `<worktree>/cli/` to PATH, and execs `claude --plugin-dir <worktree>/plugins/lets`. Each invocation is self-contained — no global install, no marketplace mutation.
- **`make dev-tmux`** auto-discovers `.worktrees/*/` and spawns a tmux session with one Claude pane per worktree. Pass `WORKTREES="name1 name2"` to limit. Threshold prompt at >10.
- Implementation: `scripts/dev/run.sh` (subcommands: `build|info|claude|tmux`). The Makefile targets are thin shims for discoverability via `make help`.

**Gotcha — Claude-inside-Claude.** `make dev` must run from a **host terminal**, not from a Bash tool inside an existing Claude session. The Bash tool spawns a subshell that `exec`s the inner `claude`; the outer Claude's tool harness captures stdin/stdout, so the inner Claude has no terminal to interact through and hangs. Symptom: the Bash tool times out with no visible Claude prompt. Use a fresh tmux pane or terminal window instead.

The remaining dev-binary gotchas — `LETS_ENV_VERSION` stamping, old worktrees, PATH-shadowing ("trust the branch"), and `IsDev()` semantics — live in `CONTRIBUTING.md` "### Dev binary: `make dev` / `make dev-tmux`".

## Key Concepts

> Path convention: paths like `commands/`, `skills/`, `rules/lets-rules.md` in this doc are **relative to `plugins/lets/`** (the plugin root, also exposed as `${CLAUDE_PLUGIN_ROOT}` at runtime). Paths starting with `scripts/release/`, `scripts/remote/`, `docs/`, `reference/` are relative to the **repo root** (outside the plugin payload).
>
> Go CLI source paths (`cli/cmd/lets/main.go`, `cli/internal/...`) are relative to the **repo root**. The Go module root is `cli/` - all `go` commands operate from there (or via the repo-root `Makefile`).

- **Commands** = user-initiated workflows (sessions, commits, reviews)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [restarter/lets-workflow](https://github.com/restarter/lets-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
