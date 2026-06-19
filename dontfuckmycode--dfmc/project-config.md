---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

## Project

**DFMC** ("Don't Fuck My Code") is a single-binary Go code intelligence assistant. It combines local analysis (AST, codemap, security heuristics) with a multi-provider LLM router and an offline fallback. CLI, Bubble Tea TUI, and embedded Web API all drive the same `internal/engine.Engine`.

Module: `github.com/dontfuckmycode/dfmc`. Go 1.25.

## Build, test, lint

Only the `Makefile`'s `clean` (`rmdir /s /q`) and `VERSION` (`NUL`) recipes are Windows-bound; `make test`, `test-race`, `lint`, `vuln`, and `security` shell out to `go`/tooling and work anywhere. Prefer direct `go` commands for narrow loops:

```bash
CGO_ENABLED=1 go build -o bin/dfmc.exe ./cmd/dfmc
CGO_ENABLED=1 go test -race -count=1 ./...   # == make test-race
go test ./internal/engine/...
go test ./internal/engine -run TestAgentLoop -v
go vet ./...
gofmt -w $(git ls-files '*.go')
staticcheck ./...
make lint       # vet + staticcheck + golangci-lint
make security   # govulncheck + gosec
```

**CGO matters.** Tree-sitter bindings require CGO. With `CGO_ENABLED=0`, the build can still pass but AST falls back to regex (`internal/ast/backend_stub.go`); `dfmc status` / `dfmc doctor` report `ast_backend: regex`. On Windows, `CGO_ENABLED=1` and `go test -race` require `gcc` on `PATH`.

## Architecture

### Engine is the hub

`internal/engine.Engine` is constructed in `cmd/dfmc/main.go`, owns the main subsystems, and is shared by all UIs. `engine.go` handles construction/lifecycle/state; most methods live in `engine_<topic>.go` siblings. Grep `func (e *Engine)` before editing an Engine method.

Load-bearing Engine files:

- `engine_tools.go` — `CallTool` / `CallToolFromSource` outer wrapper and `tool:error` / `tool:complete` events.
- `engine_tools_lifecycle.go` — `executeToolWithLifecycle`, the required safety funnel for tool calls: subagent/skill/path gates, approval, hooks, panic guard, timeout/denial/panic events, and shared `Event.Seq` stamping. New tool entry points must route through this or `CallTool` unless a documented exception applies.
- `engine_meta_hooks.go` — unwraps `tool_call` / `tool_batch_call` so hooks and side effects reach inner backend tools.
- `engine_context.go` — context budget, recommendations, chunking, reserve breakdown.
- `engine_prompt.go` — system prompt and prompt runtime/recommendation logic.
- `engine_ask.go`, `engine_ask_stream.go`, `engine_ask_history*.go` — Ask paths, streaming, history trimming.
- `engine_intent.go` — builds intent `Snapshot` and calls `Intent.Evaluate` before Ask.
- `engine_passthrough.go` — status, memory, conversation, provider passthroughs.
- `engine_analyze.go` — analysis, dead-code/complexity passes, text strippers.

Tool lifecycle events share a monotonic `Event.Seq` from `Engine.allocToolEventSeq()`; subscribers dedupe failure telemetry on `(Type, Seq)`.

### Engine-owned subsystems

- `internal/ast` — tree-sitter with CGO, regex fallback otherwise.
- `internal/codemap` — symbol/dependency graph, cycles, hotspots, path traversal, DOT/SVG.
- `internal/context` — ranks/compresses snippets under token budget; every token sent should be justified.
- `internal/provider` — primary + fallback router. Offline provider is always registered; missing keys create graceful placeholders. Protocols: Anthropic, OpenAI, OpenAI-compatible.
- `internal/tools` — backend registry plus four meta tools: `tool_search`, `tool_help`, `tool_call`, `tool_batch_call`. Tool loop lives around `agent_loop_native.go`; parking in `agent_parking.go`; lifecycle in `engine_tools_lifecycle.go`.
- `internal/memory`, `internal/conversation`, `internal/storage` — bbolt-backed memory, JSONL conversations, store handle. A second DFMC process hits `ErrStoreLocked`; degraded commands like `doctor` still run.
- `internal/hooks` — best-effort lifecycle shell hooks; failures log but do not block.
- `internal/taskstore` — persisted `supervisor.Task` store for TODO, HTTP, and MCP APIs.
- `internal/langintel` — embedded per-language knowledge bases (Go/Java/C#/...) surfaced through engine init and `ui/web/server_langintel.go`.
- `internal/bot` — Telegram bridge wired into the Engine; controlled from `cmd/dfmc/main.go`/`startup_args.go`, the TUI `telegram_panel.go`, and web/help surfaces.
- `EventBus` — shared fan-out for TUI, web SSE, remote control.

### Tooling rules worth preserving

Context gathering order is cheapest to most precise: `grep_codebase` → `codemap` → `find_symbol` → `read_file`.

`find_symbol` is language-aware and returns full scope; `parent` disambiguates receivers/classes. `codemap` is a signatures-only outline and should generally be used once per session, not per file.

`read_file` defaults to a 200-line window and reports `total_lines`, `returned_lines`, `truncated`, and `language`. `truncated` means returned lines are fewer than total lines, even if the caller intentionally requested a slice.

Tool validation should use `missingParamError` for required params. `normalizeToolParams` handles common aliases (`edit_file` `old`/`new`; `write_file` `text`/`body`/`data`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DontFuckMyCode/dfmc](https://github.com/DontFuckMyCode/dfmc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
