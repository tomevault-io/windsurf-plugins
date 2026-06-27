---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`agentscope-go` is a Go port of the Python [AgentScope](https://github.com/agentscope-ai/agentscope) multi-agent LLM framework. The module path is `github.com/alanfokco/agentscope-go`. All library code lives under `pkg/agentscope/`; runnable demos live under `examples/`.

Python reference code is at `/Users/alanfokco/Github/agentscope/` (main branch). When adding features, check the Python implementation first for design consistency.

`go.mod` declares `go 1.25.0`, but `CONTRIBUTING.md` asks contributors to keep the code compatible with **Go 1.22+**. Don't reach for `>=1.23` standard-library features without checking.

## Common commands

```bash
# Build everything (library + every example main).
go build ./...
go build ./examples/...

# Static checks and tests (matches CI in .github/workflows/ci.yml).
go vet ./...
go test ./...

# Run a single package's tests, or a single test:
go test ./pkg/agentscope/pipeline -run TestName -v

# Run any example (each has its own main package).
go run ./examples/simple
go run ./examples/agent_v2
go run ./examples/streaming
go run ./examples/middleware
go run ./examples/react_tool
go run ./examples/react_builtin_tools
go run ./examples/multi_provider
go run ./examples/structured_output
go run ./examples/pipeline_multi_agent
go run ./examples/permission
go run ./examples/agent_team
go run ./examples/mcp
go run ./examples/embedding
go run ./examples/long_term_memory
go run ./examples/rag_react
go run ./examples/a2a_http
go run ./examples/realtime_echo
go run ./examples/agent_service
go run ./examples/scheduled_task
go run ./examples/model_call
go run ./examples/multimodal
go run ./examples/multiagent
```

`vendor/` is checked into the working tree (and listed in `.gitignore` — i.e. it's a local convenience, not the source of truth). If you change dependencies, run `go mod tidy` and don't commit a stale `vendor/`.

LLM-backed examples need one of: `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, or `DASHSCOPE_API_KEY` (+ optional `DASHSCOPE_BASE_URL`). The `loadChatModelFromEnv` helpers inside the examples pick a backend in the order Anthropic → DashScope → OpenAI.

## Deployment

Code must be rsync'd to `root@builder:/opt/Projects/agentscope-go` for commit and push. Do NOT commit or push locally — always rsync first, then `git add/commit/push` on builder.

## Architecture

The package layout intentionally mirrors the Python project (see `docs/migration_from_python.md` for the full mapping). Each subpackage exposes a small interface plus one or more concrete implementations:

### Core

- **`config.go`** — global `Init(opts ...Option)` sets up a process-wide `Config`. `agentscope.Log()` (logrus) is the canonical logger.
- **`message`** — `Msg` with typed `ContentBlock` variants: `TextBlock`, `ThinkingBlock` (with `Extra` for provider-specific fields like Anthropic `signature`), `ToolCallBlock` (with `Extra` for OpenAI Response `call_id`), `ToolResultBlock` (with `Metadata`), `DataBlock` (Base64Source/URLSource for images/audio/video), `HintBlock` (polymorphic `Hint` — `string` or `[]ContentBlock`). `NewMsg` panics on invalid content type by design.
- **`event`** — Full event lifecycle: ReplyStart/End, ModelCallStart/End, TextBlock/ThinkingBlock/DataBlock Start/Delta/End, ToolCall Start/Delta/End, ToolResult Start/TextDelta/DataDelta/End, HintBlock, HITL events (RequireUserConfirm, UserConfirmResult, RequireExternalExecution, ExternalExecutionResult), ExceedMaxIters, Custom.

### Agent

- **`agent`** — `Agent` interface (`ID`, `Reply`, `Observe`, `Interrupt`, `SetConsoleOutputEnabled`) and `AgentBase` (UUID identity, console printing, msghub subscriptions, hooks). Two agent generations:
  - **`UnifiedAgent`** (v2) — aligns with Python's single `Agent` class. Native tool calling, streaming via `ReplyStream()` returning `<-chan event.Event`, middleware chain, permission engine, context compression, skill instructions injection, audio block filtering. Options: `WithToolkit`, `WithMiddlewares`, `WithContextConfig`, `WithPermissionContext`, `WithSkills`, `WithReadCache`.
  - **`ReActAgent`** (v1, legacy) — JSON-based tool calling protocol. Supports RAG via `WithKnowledge(...)` and basic compression via `WithCompression`.
  - **`A2AAgent`** — remote agent proxy via `a2a.Client`.
  - **`UserAgent`** — human input agent with pluggable `InputProvider`.

### Model

- **`model`** — `ChatModel` interface: `Chat`, `ChatStream` (`<-chan ChatResponse`), `CountTokens`. 9 provider adapters: `openai.go`, `anthropic.go`, `dashscope.go`, `deepseek.go`, `gemini.go`, `moonshot.go`, `ollama.go`, `xai.go`, `openai_response.go`. All share `internal/httpx` for HTTP calls.
  - **Call options**: `WithTemperature`, `WithMaxTokens`, `WithTools`, `WithToolChoice`, `WithThinking(enable, budget)`, `WithReasoningEffort(effort)`, `WithRetries(max, delay)`.
  - **`ChatUsage`** tracks `InputTokens`, `OutputTokens`, `CacheCreationInputTokens`, `CacheInputTokens`.
  - **`FallbackChatModel`** — automatic primary → fallback failover.
  - **`SecretStr`** — wrapper type that redacts API keys in `String()`/`MarshalJSON()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlanFokCo/agentscope-go](https://github.com/AlanFokCo/agentscope-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
