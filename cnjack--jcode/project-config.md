---
trigger: always_on
description: Go CLI coding assistant — [Eino](https://github.com/cloudwego/eino) + BubbleTea v2 TUI + Vue 3 web UI.
---

# AGENTS.md — JCode Project Development Guide

Go CLI coding assistant — [Eino](https://github.com/cloudwego/eino) + BubbleTea v2 TUI + Vue 3 web UI.

- **Module:** `github.com/cnjack/jcode` | **Entry:** `cmd/jcode/` | **Config dir:** `~/.jcode/`

---

## Quick Start

```bash
make build        # generate → build-web → go build
make install      # generate → build-web → go install
make run          # go run ./cmd/jcode/
make lint         # golangci-lint + eslint/oxlint
make doctor       # system check
```

- `make build-web` requires `pnpm`. Frontend builds to `internal/web/dist/`.
- `make generate` runs `go generate ./internal/model/...` — fetches models.dev data and generates `internal/model/registry_generated.go`. **Do NOT manually edit that file.**
- Build injects `Version`, `BuildTime`, `GitCommit` via ldflags into `internal/command`.

---

## Architecture Overview

```
cmd/jcode/           # Entry: subcommands (mcp, acp, web), main event loop
internal/
  command/           # Subcommand implementations + interactive session orchestration
  agent/             # ChatModelAgent factory + middleware chain
  runner/            # Agent run loop + event bus
  handler/           # AgentEventHandler interface (TUI / ACP / Web implementations)
  tools/             # All built-in tools + Executor/Env abstraction
  model/             # OpenAI-compatible chat model + model registry (build-time generated)
  config/            # JSON config loader + logger
  prompts/           # System/plan prompts + AGENTS.md injection + env info
  session/           # JSONL session recording/replay
  skills/            # Skill loader (builtin → user → project override chain)
  team/              # Multi-agent team coordination
  tui/               # BubbleTea v2 TUI components
  web/               # HTTP server (REST + SSE + PTY) + embedded Vue dist
web/                 # Vue 3 + Vite + TypeScript frontend source
script/              # Build-time code generation
```

### Key Design Decisions

- **Three transports, one interface:** TUI, ACP (JSON-RPC), and Web all implement `AgentEventHandler`. New transports only need to implement this interface.
- **Middleware chain in `agent/`:** Ordered outermost→innermost: langfuse → budget → compaction → recovery → approval. **Approval is always innermost** — never add middleware after it.
- **Tools are methods on `*Env`:** Each tool is created via `env.NewXxxTool()`, receives the shared `Env` for file I/O and command execution. This enables transparent local/SSH switching.
- **Eino framework:** We use [cloudwego/eino](https://github.com/cloudwego/eino) `adk.ChatModelAgent` — not raw LLM calls. Follow Eino's `tool.InvokableTool` + `schema.ToolInfo` patterns.

---

## Conventions (MUST follow)

### Logging & Output
- **All diagnostics go to `config.Logger()`** (writes to `~/.jcode/debug.log`). Never use `fmt.Print`, `log.Print`, or write to stdout/stderr directly — the TUI owns stdout.
- Tool execution errors are returned as plain strings (the agent reads them). Do NOT `panic` or `log.Fatal` in tool code.
- Exclude the script/ directory from the linter — it contains code generation scripts that may not follow all conventions.

### Error Handling
- Tools return `(string, error)`. Return descriptive error strings that help the agent self-correct. Include file paths, line numbers, or command output in error messages.
- Use `fmt.Errorf("tool_name: %w", err)` for wrapped errors in non-tool code.

### File Paths
- All file paths in tools must be resolved via `env.ResolvePath(path)`. This handles relative→absolute conversion and logs warnings for paths escaping the working directory.
- Store and pass absolute paths internally. Only accept relative paths at the tool input boundary.

### Tool Development Pattern
1. Define `XxxInput` struct with `json` tags
2. Create `func (e *Env) NewXxxTool() tool.InvokableTool` on `*Env`
3. Build `schema.ToolInfo` with `schema.NewParamsOneOfByParams(...)` — use `schema.String`, `schema.Integer`, `schema.Boolean`, `schema.Array`
4. Register in `buildAllTools()` in `internal/command/interactive.go` (and `acp.go`, `web.go`)
5. If the tool is read-only, also add to `buildPlanTools()`
6. **Approval policy:** Read-only tools skip approval. Mutating tools require approval unless `AutoApprove` is set. Match existing patterns in `approval.go`.

### Approval Policy
- Read-only tools (read, grep, glob, todoread, etc.): auto-approved
- Mutating tools (edit, write, execute): require user approval
- Execute exceptions: background commands and safe prefixes (`ls`, `cat`, `echo`, `which`, `git status`, `git log`, etc.) are auto-approved
- `switch_env`: always requires approval

### Code Style
- Follow standard Go conventions. The linter config (`.golangci.yml`) enforces: `errcheck`, `govet`, `staticcheck`, `unused`, `revive`, `gocritic`, `funlen` (max 800 lines/600 statements).
- Use `context.Context` as the first parameter. Thread cancellation properly.
- Prefer returning errors over panicking. Only `panic` for truly unrecoverable programmer errors.
- Interfaces live in the package that consumes them (e.g., `AgentEventHandler` in `handler/`, `Executor` in `tools/`).

### Concurrency

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cnjack/jcode](https://github.com/cnjack/jcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
