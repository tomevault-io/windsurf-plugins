---
trigger: always_on
description: Guidance for Claude Code working in this repository. The octo-agent equivalent is `.octorules`.
---

# CLAUDE.md

Guidance for Claude Code working in this repository. The octo-agent equivalent is `.octorules`.

## Project

`octo-agent` — a Go 1.22+ AI agent CLI distributed as a single binary. Module path: `github.com/open-octo/octo-agent`. Ships as CLI + embedded Web UI + IM bridges via `octo serve`. Per-feature design notes live under `dev-docs/`.

## Commands

```bash
make build                                                  # ./octo (or set VERSION=0.x.y for releases)
make test                                                   # go test -race ./...
make vet                                                    # go vet ./...
make fmt-check                                              # gofmt -l . must print nothing
make fmt                                                    # gofmt -w .
make tidy                                                   # go mod tidy

go test ./internal/agent/                                   # single package
go test ./internal/provider/anthropic/ -run TestSendStream  # single test
go test -race -v ./internal/tools/                          # verbose race
```

## Architecture

Five-layer stack with one-directional dependencies:

1. **CLI (`cmd/octo/`)** — entry point (`main.go`), flag parsing, REPL loop (`repl.go`), session resume/list flags, slash-command dispatch, output streaming. Reaches the LLM through `internal/app` rather than importing `provider` directly.

2. **App bootstrap (`internal/app/`)** — the single place that constructs provider clients and adapts them to `agent.Sender`. Every entry point (`cmd/octo`, `internal/server`, IM channels) reaches the LLM through it rather than importing `provider` directly. Also owns the permission gate, sub-agent spawner, and `WireTools` unification.

3. **Agent core (`internal/agent/`)** — the loop, plus everything stateful:
   - `agent.go` — `Agent`, `Turn`, `TurnStream`, `Run`, `RunStream`. History rollback on error.
   - `history.go` — message log; goroutine-safe.
   - `content.go` — `ContentBlock` union (text / tool_use / tool_result). `Message.Blocks` overrides `Message.Content` when set; nil falls back to plain string for backward-compatible session JSON.
   - `session.go` — JSON persistence under `~/.octo/sessions/`.
   - `tool.go` — `ToolDefinition`, `ToolExecutor` interfaces.
   - `Sender` interface stack: `Sender` → `StreamingSender` → `ToolSender` → `ToolStreamingSender`. Each builds on the previous; type-assertion in callers picks the highest available capability.

4. **Providers (`internal/provider/`)** — per-vendor wire-format adapters. `provider.go` defines the interfaces; each subdirectory implements one protocol:
   - `anthropic/` — Messages API. `x-api-key` + `anthropic-version` headers. `system` as top-level field. Content blocks `[{type:"text", text}]`. SSE aggregator dispatches on `message_start`/`content_block_delta`/`message_delta`. Tool calls land as `content_block_start` of type `tool_use` with subsequent `input_json_delta` deltas.
   - `openai/` — Chat Completions. `Authorization: Bearer`. `system` carried as `messages[0]`. SSE aggregator parses `chat.completion.chunk`; tolerates missing `[DONE]` sentinel (some third-party servers omit it). Tool calls arrive in `delta.tool_calls[]` with chunked JSON arguments.

   Provider wire quirks are encapsulated here — the agent layer never branches on protocol.

5. **Tools (`internal/tools/`)** — concrete `ToolExecutor` implementations.
   - `terminal.go` — current canonical example. Tool name `terminal` rather than `bash` because the implementation shells out via the platform shell — `sh -c` on macOS/Linux, PowerShell (`pwsh`, else `powershell`) on Windows — not a hard `/bin/bash` dependency. The shell is selected in one place: `shellCommand` in `sandbox.go`. The model is told which shell it's on via the environment context (`cmd/octo/envcontext.go`).
   - `DefaultRegistry` dispatches by tool name. `DefaultTools()` returns the set sent to the LLM (tools are on by default; `--no-tools` disables them).

## Adding capability

- **New provider** — implement `provider.Provider` (required) and optionally `provider.StreamingProvider`, `provider.ToolProvider`, `provider.ToolStreamingProvider`. Put it under `internal/provider/<name>/`. Each protocol's wire-format quirks are isolated inside the package; the agent layer must not learn about them.
- **New tool** — implement `agent.ToolExecutor` and `Definition() agent.ToolDefinition` returning the JSON Schema the LLM sees. Place it under `internal/tools/<name>.go`. Register it in `tools.DefaultRegistry` and add it to `tools.DefaultTools()` if it belongs in the default set.
- **New skill** — `~/.octo/skills/<name>/SKILL.md` with the same frontmatter format Claude Code uses. The skill loader composes existing tools — adding a skill should not require new tool code.

## Conventions

From `.octorules`:

- **One-directional deps.** `provider → agent` is enforced; `agent` must not import `provider`. Tests verify this implicitly by living in the same package as the code they test.
- **Test placement.** `*_test.go` siblings of source files. No external test frameworks beyond the stdlib + `httptest`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-octo/octo-agent](https://github.com/open-octo/octo-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
