---
trigger: always_on
description: <instruction_file_topology>
---

# Orqestra - Copilot Instructions

<instruction_file_topology>

## Instruction File Topology

- Canonical instruction file: `.github/copilot-instructions.md`.
- Symlink topology: `.github/copilot-instructions.md` <-- symlink <-- `CLAUDE.md`.
- `CLAUDE.md` must remain a symlink to `.github/copilot-instructions.md`. Edit the canonical target, not a divergent copy. Verify with `readlink CLAUDE.md` after instruction-file changes.

</instruction_file_topology>

<system_router>

## Routing Rules

Before planning or editing code, decide whether the task touches one of these domains and load the matching file first.

1. Terminal UI and Bubble Tea architecture: any edit under `internal/tui/`
   - Read `.github/tui-instructions.md`.
2. Agent execution, harnesses, sandboxing, token limits, or plan persistence: edits under `internal/agent/`, `internal/harness/`, `internal/sandbox/`, `internal/tokenlimit/`, or `internal/plan/`
   - Read `.github/agent-instructions.md`.
3. Cross-domain changes
   - Read every matching instruction file before designing the change.

</system_router>

<commands>

## Commands

- `make build` — builds `./bin/orqestra` (CGO disabled, stripped).
- `make test` — unit tests with race detector and coverage. Fast, no external deps.
- `make test-integration` — adds `-tags integration`; needs `git` and `go build` in PATH.
- `make test-sandbox` — adds `-tags 'darwin integration'`; needs `sandbox-exec` (macOS only).
- `make test-e2e` — `-tags e2e` in `internal/harness/`; requires real `claude` CLI and API access.
- `make lint` — `go vet ./...`.
- Single package/test: `go test -race ./internal/agent/ -run TestArchitect_… -v`.
- Run TUI: `make run` (or `./bin/orqestra` after `make build`).
- Headless smoke: `./bin/orqestra --prompt "..." --auto-approve --config orqestra.yaml`.
- CLI subcommands: `plan`, `validate`, `exec`, `usage`, `reset-usage`, plus internal `mcp-bridge` (invoked by Claude CLI as an MCP server, not by users).

</commands>

<repo_truth>

## Hard Architecture Facts

- Orqestra is a macOS-first Go CLI/TUI that orchestrates Claude Code through harnesses, not direct model APIs.
- The active pipeline in `internal/orchestrator/` is: Researcher -> Architect -> Critic -> human plan gate -> sandboxed Worker -> worker self-validation -> optional worktree commit/merge.
- The current plan contract is `agent.RawPlan`: raw markdown read from Claude plan files by `agent.ReadPlanFromRun`. `agent.Specification`, `agent.PlanOutput`, `agent.ProjectPlan`, and `internal/scheduler/` are legacy or secondary paths unless the task explicitly targets them.
- `internal/harness/` owns CLI invocation, Claude stream parsing, session IDs, plan-file paths, and token usage. Agent domain structs must not import harness types only to carry execution metadata.
- Worker execution must go through the macOS seatbelt sandbox in `internal/sandbox/` via `harness.SandboxCLIRunner`; repo writes should be isolated in a per-run worktree when available.
- `internal/tui/` is Bubble Tea MVU. `Update()` mutates model state and returns commands; `View()` renders only.
- Session artifacts live under `.orqestra/sessions/<run>/`; Claude session logs are copied there for diagnostics when available.
- Configuration is YAML loaded through `internal/config/`, with embedded defaults from `internal/config/pipeline.yaml`. Explicit user-supplied paths, model refs, sandbox settings, and prompt files must fail clearly when invalid.

</repo_truth>

<architectural_fault_lines>

## Architectural Fault Lines To Hedge

- Plan extraction depends on external Claude plan files and JSONL session logs. Missing session IDs, unreadable plan files, empty plans, and paths outside `~/.claude/plans/` are integrity failures, not soft warnings. Directory-scan fallback is allowed only behind the existing security gate and must be logged with enough context to debug the source of truth.
- The orchestrator is a long stateful pipeline with intentional best-effort diagnostics. Keep a hard boundary between integrity errors and optional observability: config/model resolution, sandbox setup, plan extraction, worker execution, validation verdict parsing, and merge conflict state must return, emit, or gate; session-log copy, plan-history diff, and commit-message generation may warn and continue if the user-visible state remains truthful.
- Worktree isolation currently falls back to writable-repo execution when branch detection or worktree creation fails. Changes in execution, merge, or sandbox code must either remove that fallback, gate it explicitly, or prove with tests and user-visible events that the fallback cannot hide unsafe writes.
- `internal/scheduler/` uses an opaque `spec any` payload and mutates per-agent status from goroutines. Treat it as legacy/experimental unless requested. Any new scheduler work needs typed payload boundaries or explicit adapters, race-safe status/event handling, unknown-dependency checks, cycle tests, and `go test -race` coverage.
- Config model lookup currently accepts case-insensitive aliases in some paths. Do not add new fallback resolution. Tightening this behavior requires migration-aware tests for exact refs, wrong-case refs, missing refs, and graph validator refs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aka-rider/orqestra](https://github.com/aka-rider/orqestra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
