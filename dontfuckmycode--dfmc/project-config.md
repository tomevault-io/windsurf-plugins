---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**DFMC** ("Don't Fuck My Code") — a code intelligence assistant distributed as a single Go binary. It combines local code analysis (AST + codemap + security heuristics) with a multi-provider LLM router that falls back to an offline provider when API keys are missing or calls fail. Three UIs (CLI, bubbletea TUI, embedded Web API) all drive the same `engine.Engine`.

Module path: `github.com/dontfuckmycode/dfmc`. Go 1.25.

## Build, test, lint

The `Makefile` is Windows-oriented (uses `NUL`, `rmdir /s /q`). Prefer invoking `go` directly in bash:

```bash
# Build (CGO required for tree-sitter AST — see below)
CGO_ENABLED=1 go build -o bin/dfmc.exe ./cmd/dfmc

# Fast dev run (no binary)
go run ./cmd/dfmc <command> [args]

# Full test suite (what Makefile uses)
CGO_ENABLED=1 go test -race -count=1 ./...

# Single package / single test
go test ./internal/engine/...
go test ./internal/engine -run TestAgentLoop -v

# Lint / format
go vet ./...
gofmt -w .
```

**CGO matters.** Tree-sitter bindings (`tree-sitter-go`, `-javascript`, `-typescript`, `-python`) require CGO. With `CGO_ENABLED=0` the build still succeeds but AST silently falls back to the regex extractor in `internal/ast/backend_stub.go`, and `dfmc status` / `dfmc doctor` will report `ast_backend: regex`. If AST behavior looks wrong, check the backend before blaming the code.

## Architecture

### Engine is the hub

`internal/engine.Engine` (constructed in [cmd/dfmc/main.go](cmd/dfmc/main.go)) owns every subsystem and is passed by pointer into all three UIs.

The Engine type itself is split by domain across sibling files; [engine.go](internal/engine/engine.go) keeps construction/lifecycle/state and the rest live in:

- [engine_tools.go](internal/engine/engine_tools.go) — `CallTool`, panic-guarded execution, approval/hooks lifecycle. **Every tool call — user-initiated (`CallTool`) and agent-initiated (agent loop / subagent) — funnels through `executeToolWithLifecycle`**, which owns the approval gate, pre/post hook dispatch, and the panic guard. When adding a new tool surface, route through this helper rather than calling `tools.Engine.Execute` directly, or approval/hooks will silently bypass it.
- [engine_context.go](internal/engine/engine_context.go) — context budget/recommendations/tuning + chunk building + reserve breakdown
- [engine_prompt.go](internal/engine/engine_prompt.go) — `buildSystemPrompt`, `PromptRecommendation*`, `promptRuntime*`
- [engine_ask.go](internal/engine/engine_ask.go) — `Ask`, `AskRaced`, `AskWithMetadata`, `StreamAsk`, history trimming
- [engine_intent.go](internal/engine/engine_intent.go) — glue for the intent router: builds the Snapshot from engine state and runs `Intent.Evaluate` before each Ask
- [engine_passthrough.go](internal/engine/engine_passthrough.go) — `Status`, memory/conversation/provider passthrough surface
- [engine_analyze.go](internal/engine/engine_analyze.go) — `AnalyzeWithOptions`, dead-code/complexity passes, text strippers

Subsystems owned by the Engine:

- `AST` ([internal/ast](internal/ast/)) — tree-sitter when CGO is on, regex fallback otherwise. Parse metrics are tracked per-call.
- `CodeMap` ([internal/codemap](internal/codemap/)) — symbol/dependency graph built on top of AST; supports cycles, hotspots, path traversal, DOT/SVG export.
- `Context` ([internal/context/manager.go](internal/context/manager.go)) — ranks and compresses file snippets under a token budget before the LLM sees them. Core design principle: **every token sent is justified**.
- `Providers` ([internal/provider/router.go](internal/provider/router.go)) — router with a primary + fallback list. The offline provider is always registered; missing API keys yield a `PlaceholderProvider` that degrades gracefully instead of erroring. Protocols: `anthropic`, `openai`, `openai-compatible` (covers deepseek/kimi/zai/alibaba/generic/ollama).
- `Tools` ([internal/tools](internal/tools/)) — backend registry: file ops (`read_file`, `write_file`, `edit_file`, `apply_patch`, `list_dir`), search (`grep_codebase`, `glob`, `find_symbol`, `codemap`, `ast_query`), shell (`run_command`), git (`git_status`/`_diff`/`_log`/`_blame`/`_branch`/`_commit`/`_worktree_*`), web (`web_fetch`, `web_search`), planning (`task_split`, `orchestrate`, `delegate_task`), reasoning (`think`, `todo_write`). Tool-capable providers see the four meta tools (`tool_search`/`tool_help`/`tool_call`/`tool_batch_call`) and dispatch backend tools through them — this keeps the model's tool list short and the protocol stable across providers. Bounded loop in [internal/engine/agent_loop_native.go](internal/engine/agent_loop_native.go); park-and-resume in [agent_parking.go](internal/engine/agent_parking.go); per-tool execution + lifecycle in [engine_tools.go](internal/engine/engine_tools.go).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DontFuckMyCode/dfmc](https://github.com/DontFuckMyCode/dfmc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
