---
trigger: always_on
description: `AGENTS.md` is a symlink to this file. Keep this file short, durable, and
---

# tclaude agent instructions

`AGENTS.md` is a symlink to this file. Keep this file short, durable, and
useful as startup context for coding agents. Do not use it as a changelog,
implementation diary, roadmap, or project-management scratchpad.

## What tclaude is

`tclaude` is a Go CLI that wraps agentic coding harnesses in tmux and adds
session management, conversation search, usage/status reporting, worktree
helpers, and multi-agent coordination.

The project is harness-agnostic. Four harnesses are registered via
`--harness claude|codex|opencode|copilot` (Claude Code is the default); the
selected harness is persisted per conversation. The harness seam lives in
`pkg/claude/harness`.

Many identifiers still contain historical `Claude`/`claude`/`TCLAUDE_` names
even when the code is now harness-agnostic. Treat those names as historical,
not Claude-Code-only. Do not opportunistically mass-rename them; only rename at
a clean, contained rewrite point.

## Build and test

Common commands:

```bash
go build ./...
go test ./...
go test ./pkg/claude/conv/...
golangci-lint run ./...
go install . ./cmd/...
```

The repo builds two binaries: a bare `go install .` gets you only `tclaude`,
while `. ./cmd/...` adds the standalone `tclaude-agentd` daemon. See
`CONTRIBUTING.md` for why not `./...`.

CI runs `go test ./...` and `golangci-lint run ./...`. Do make sure your changes at 
least build, and run focused local tests when they help your own iteration on the code 
you are changing.

Platform target: Linux and macOS. WSL is treated as Linux for practical use.
Native Windows is not a supported development target; do not design new
features around native Windows behavior unless the operator explicitly asks.

## Where to look

- Entry points: `main.go` (the `tclaude` CLI, via `pkg/claude.Cmd()`) and
  `cmd/tclaude-agentd/main.go` (the standalone daemon, via
  `agentd.RootCmd()`). Both go through `pkg/claude/cli`, which owns the
  process-level entry sequence and the shared root-command wiring — put
  anything both binaries need there rather than duplicating it.
- Root command wiring: `pkg/claude/claude.go`.
- Harness design and capability matrix: `docs/harnesses.md`.
- Adding another harness: `docs/adding-a-harness.md`.
- Agent coordination: `docs/agents-and-groups.md` and
  `docs/spawning-and-lifecycle.md`.
- Dashboard: `docs/dashboard.md`.
- Sessions, conversations, worktrees, tasks, status bar, notifications:
  corresponding files under `docs/`.
- Flow-test helpers and simulators: `pkg/testharness/`.
- Contributing and flow-test style: `CONTRIBUTING.md`.

Avoid maintaining exhaustive package inventories here. They drift quickly; use
the code tree and focused docs as the source of truth.

## Architecture guardrails

- Commands use Cobra through Boa (`boa.CmdT[...]`) with
  `common.DefaultParamEnricher()` unless nearby code establishes another
  pattern.
- Session and conversation state lives in SQLite under
  `~/.tclaude/data/db.sqlite`. The sibling `~/.tclaude/api/` tree is reserved
  for the agent-reachable daemon socket; private daemon state stays under
  `~/.tclaude/data/`.
  Legacy JSON files may still be written for compatibility, but SQLite is the
  source of truth for tclaude.
- Harness support is capability-based. Callers should gate behavior on the
  harness descriptor (`Supports*` / `Can*`) and degrade gracefully when a
  contract is absent.
- In-pane slash-command delivery via tmux `send-keys` is an injection sink.
  Lifecycle command tokens must be compile-time constants from the harness
  lifecycle, never interpolated user input. User-controlled titles or text sent
  toward these paths must pass the existing charset/length gates.
- Platform-specific code should use Go build tags such as `_linux.go`,
  `_darwin.go`, and `_unix.go`. Treat old `_windows.go` files as vestigial
  unless the task explicitly concerns them.

## Testing guidance

Unit tests live next to the code they cover. Flow tests live under
`pkg/claude/agentd/*_flow_test.go` and run under plain `go test ./...`.

Flow tests exercise production paths through the daemon HTTP mux. Only the
external subprocess boundaries are swapped:

- `clcommon.Default` for tmux.
- `agentd.Spawn` for `tclaude session new`.
- `agentd.runPluginShell` for plugin shell execution when needed.
- `session.SetCodexEffectiveConfigProbeForTest` for the `codex app-server`
  effective-config read that resolves a filtered Codex launch's provider route.
  `agentd`'s `TestMain` installs a binary-wide default, so only tests needing a
  specific provider route swap their own.

Keep new tests focused on user-visible surfaces such as CLI/API results,
conversation listings, and dashboard snapshots. Avoid asserting on simulator
internals when a production read path can be exercised instead.

For manual dashboard visual smoke, first check that Linux-side Chrome/Chromium
exists, then run:

```bash
TCLAUDE_DASHSNAP=1 TCLAUDE_DASHSNAP_SHARD=1/4 go test ./pkg/claude/agentd/ -run TestDashSnap -v -count=1 -timeout 600s
```

Run shards `1/4` through `4/4` to cover the full matrix (each takes a few
minutes; an unsharded full run needs `-timeout 1800s`). See "Visual smoke
testing" in `docs/dashboard.md`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tofutools/tclaude](https://github.com/tofutools/tclaude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
