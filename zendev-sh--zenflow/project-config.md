---
trigger: always_on
description: Instructions for AI contributors working on the zenflow codebase.
---

# AGENTS.md - zenflow

Instructions for AI contributors working on the zenflow codebase.

## Commands

```bash
go build ./...                  # Build library
go build ./cmd/zenflow/         # Build CLI
go test ./...                   # Test all packages
go test -cover ./...            # Test with coverage
golangci-lint run               # Lint
./scripts/smoke-examples.sh     # Vet + build all 18 embedding examples
go test -tags e2e ./...         # Real-LLM integration tests (needs API keys)
```

## Architecture

zenflow is a declarative multi-agent workflow engine for Go. Workflows
are YAML; the engine is a DAG executor with an LLM coordinator that
narrates progress and routes events through hub-and-spoke mailboxes.
The Go SDK layer is [goai](https://goai.sh) - any provider goai
supports works as a zenflow agent backend.

```
zenflow/
├── doc.go                      # Package zenflow doc.
├── interfaces.go               # Storage/Tracer/StepIsolation/ApprovalHandler aliases (root facade)
├── workflow.go                 # Workflow / Step / StepResult / WorkflowResult / Run aliases (root facade)
├── duration.go                 # Duration alias + FormatDuration / ParseDurationStrict re-exports
├── router_facade.go            # Re-exports for internal/router/ public API
├── transcript_facade.go        # Re-exports for internal/resume/ public API
├── coord_facade.go             # Re-exports for internal/coord/ tool factories
├── agent_facade.go             # Re-exports for internal/exec/ AgentRunner ecosystem (AgentRunner, AgentResult, 22 WithRunner*, AgentHandle, sentinels)
├── orchestrator_facade.go      # Re-exports for internal/exec/ Orchestrator + 49 With* + Executor + Storage backends + parsers + coord factory + JSON coordinator + ~60 utility symbols
├── internal/
│   ├── types/                  # Event, EventType, MessageKind, Output, ProgressSink, PermissionHandler/Request (leaf, no deps on root)
│   ├── spec/                   # Workflow / Step / Run / StepResult / AgentConfig / Duration types + Storage / Tracer / StepIsolation / ApprovalHandler / ModelResolver interface contracts + parser & validator helpers
│   ├── router/                 # MessageRouter, MailboxStore, deliveryEngine (race-safe send/wake)
│   ├── resume/                 # TranscriptStore, InMemoryTranscriptStore
│   ├── coord/                  # RunnerHandle interface + 4 coord goai.Tool factories (forward_to_agent, send_message, narrate, finalize)
│   └── exec/                   # AgentRunner + Executor + Orchestrator + JSON coordinator + RunFlow/RunGoal/RunAgent + ResumeFlow + 49 With* options + Storage backends (MemoryStorage, FileStorage) + SharedMemory + parsers + validators + scheduler + CEL evaluator + portability lints + isolation default + lifecycle + prompt assembly + 16 source files moved from root
├── cmd/zenflow/
│   ├── main.go                 # CLI entrypoint
│   ├── flags.go                # Shared flag parser
│   ├── cmd_*.go                # Subcommand handlers (flow, goal, agent, validate, plan)
│   ├── orchestrator_opts.go    # Build orchestrator from CLI flags
│   ├── provider.go             # Provider resolver
│   ├── thinking.go             # --thinking flag
│   ├── trace_otel.go           # --trace flag
│   ├── signals.go              # Signal handling
│   ├── workdir.go              # Working directory helper
│   ├── permission.go           # Interactive permission gate
│   ├── stdout_sink.go          # CLI human-readable sink
│   ├── color.go                # ANSI color helpers (auto-detect TTY)
│   ├── dag/                    # DAG plan helpers
│   └── tool/                   # CLI-only IO tools (bash/read/write/glob/grep)
├── sink/
│   ├── json.go                 # NDJSON event stream
│   ├── buffered.go             # ClosableProgressSink wrapping any sink with bounded queue
│   └── lifecycle.go            # Sink lifecycle helpers (graceful shutdown / Close())
├── examples/                   # 18 //go:build example mains
└── spec/v1/
    ├── schema.json             # JSON Schema (workflow validation)
    ├── spec.md                 # Authoritative YAML specification
    ├── examples/               # 19 reference workflows
    └── testcases/              # Conformance fixtures
```

The goai SDK is consumed directly by `internal/exec/executor.go` / `internal/exec/agent_runner.go` via `github.com/zendev-sh/goai` imports - there is no separate adapter package.

## Key rules

1. **[goai](https://goai.sh) is the LLM layer.** Never reimplement the tool loop in
   zenflow. Use `goai.GenerateText` / `goai.StreamText` with
   `WithMaxSteps` and the lifecycle hooks
   (`OnBeforeToolExecute`, `OnAfterToolExecute`, `OnBeforeStep`).
   If a [goai](https://goai.sh) capability is missing, fix it in [goai](https://goai.sh) first.
2. **Hub-and-spoke only.** Peer agents never address each other
   directly. All inter-step messages flow through the coordinator's
   `forward_to_agent` tool. Adding a peer-to-peer shortcut is a
   regression.
3. **Race-safe by construction.** Delivery uses an atomic Mailbox + Wake
   pair. Every drop has a typed reason (`DropReason`). Never swallow
   a delivery silently.
4. **YAML changes ripple in lock-step.** Update `spec/v1/spec.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zendev-sh/zenflow](https://github.com/zendev-sh/zenflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
