---
trigger: always_on
description: A Go SDK for building agents on the OpenAI Responses API. It began as a port of
---

# CLAUDE.md

## What this is

A Go SDK for building agents on the OpenAI Responses API. It began as a port of
[openai-agents-python](https://github.com/openai/openai-agents-python) and shares
its core concepts (agents, handoffs, guardrails, sessions), but **evolves
independently** — it no longer tracks upstream.

Behavior is specified in [docs/spec.md](docs/spec.md), not inherited.
[docs/migration_from_python.md](docs/migration_from_python.md) maps the two APIs
for users arriving from Python.

Module path: `github.com/zzir/agents-go` (NOT `goagents`, despite the local
directory name).

## Commands

Requires Go 1.26+.

```bash
./scripts/ci.sh                       # full CI locally: gofmt, vet, build, race tests, every submodule (GOWORK=off)
go test -race ./...                   # race detector is ON in CI — keep it green
go test -race ./agents -run TestName  # single test
go run ./cmd/verifyexamples           # every example still runs (fake model APIs)
go run ./cmd/verifydocs               # doc snippets + doc.go links name things that exist
golangci-lint run                     # CI uses golangci-lint v2.12
```

## Layout

Go workspace (`go.work`, gitignored) with twelve modules. **A submodule exists only
to keep a heavy dependency out of the core** ([spec.md §5.7](docs/spec.md)) —
anything dependency-free stays in the root module. Non-root modules `require` the
root via `replace => ..`:

- **root** — the SDK (includes `tools/bravesearch` and `models/modelkit`, the
  dependency-free toolkit + conformance suite for model adapters)
- **`mcp`** — MCP client and server (carries modelcontextprotocol/go-sdk and the
  seven indirect requirements that came with it; import path unchanged)
- **`models/anthropic`** — Anthropic Messages API backend (carries
  anthropic-sdk-go; translates to the canonical Responses format, spec §5.10)
- **`sandbox/docker`**, **`sandbox/ssh`** — sandbox backends
- **`sessions`** — SQLite/PostgreSQL `Session` backends
- **`skills`** — Agent Skills (`SKILL.md`) loader
- **`tracing/otel`** — OpenTelemetry exporter (the core stays vendor-neutral)
- **`cmd/agents-server`** — web app (REST + WS + embedded UI)
- **`examples/otel`**, **`examples/anthropic`**, **`examples/mcpserver`** — the
  examples with their own modules, for their extra deps

CI builds each module standalone with `GOWORK=off`, so a workspace-only fix can
hide a missing `go.mod` require — always validate with `./scripts/ci.sh`. The
reverse hides too: `go.work` is gitignored and CI never reads it, so a module
missing from the local `use` block drops out of `go test ./...` with no error at
all — after adding a module, run `go work use ./<module>`.

## Architecture

Core type: `agents.Agent` (a plain struct); everything orbits the runner.

- **Entry points** — `Run` returns `(RunStream, RunControl)`; `RunSync` returns
  `(*RunResult, error)`. Both go through `withMiddleware` → `runStream`, which
  takes a `rawEvents bool` for the only difference between them (whether the
  model call is streamed). Run-semantics changes are written once, in the
  `agents/run*.go` family: `run.go` holds the loop, `run_step.go` classifies a
  model response into the turn's work, `run_tools.go` executes tools (approval
  partition included), `run_handoff.go` executes handoffs, and the other
  `run_*.go` files one loop stage each (options, prepare, input guardrails,
  server cursor, persist, finish, resolve, tracing, error handlers).
- **A run executes on the consumer's goroutine.** Ranging the stream advances
  the loop; abandoning it stops the run. No producer goroutine, no context that
  must be cancelled on early exit.
- **Middleware** — `agents/middleware.go` defines `RunMiddleware`;
  `agents/middleware/` ships `Loop`, `Approval`, `Retry`, `Logging`, `Plan`,
  `Todo`. Wrapping a whole run belongs here, not in the loop.
- **Models** — `agents/model.go`; backends are `models/openai` (Responses
  API, the native format) and `models/anthropic` (Messages API, translated in
  the adapter — spec §5.10). `models/modelkit` holds the shared adapter
  plumbing and `modelkit/conformancetest` the golden matrix every `Model`
  implementation must pass. Retry / fallback / routing are provider-agnostic
  decorators (`NewRetryModel`, `NewFallbackModel`, `RouterProvider`) — never
  run-loop changes.
- **Tools** — `agents/function_tool.go`: `NewTool[Args, Result]`
  reflects Args into a strict-mode JSON schema. `agent.AsTool(...)` wraps an
  agent as a callable tool.
- **Handoffs / guardrails / HITL** — `handoff.go`, `guardrail.go` (one
  `Guardrail` type across four stages; the runner-side input gate/race lives
  in `run_input_guardrails.go`), `run_state.go`: `NeedsApproval` returns
  interruptions; serialize `RunState`, then `Approve`/`Reject` +
  `agents.ResumeRun` — runs survive process restarts.
- **Sessions** — the `agents/session` subpackage, three layers: `session.Storage`
  (reads/writes entries, knows no meaning), `session.Session` (a struct, turns
  entries into model input), and `session.Projector` (which kinds reach the
  model). The shared value types (`Source`, `ItemDisplay`, `RequestUsage`,
  `Diagnostic`, `ErrorCode`) live in session and are aliased in agents. Storage is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zzir/agents-go](https://github.com/zzir/agents-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
