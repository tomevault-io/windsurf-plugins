---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Vigolium is a high-fidelity web vulnerability scanner written in Go. It operates as a CLI scanner, REST API server with traffic ingestion, or traffic-forwarding ingestor client (`vigolium ingest`). Module: `github.com/vigolium/vigolium`, requires Go 1.26+.

## Build & Test Commands

**IMPORTANT:** Never build the binary directly with `go build` to `./vigolium` or any ad-hoc path. Always use `make build` (outputs to `bin/vigolium`) or `make install` (installs to `$GOPATH/bin`). Direct `go build` bypasses version injection and may leave stale binaries in the working directory.

```bash
make build              # Build main binary → bin/vigolium, installs to $GOPATH/bin
make test               # Run all tests (auto-installs gotestsum)
make test-unit          # Fast unit tests (-short flag, no external deps)
make test-race          # All tests with race detector
make test-e2e           # E2E tests (requires Docker, -tags=e2e)
make test-canary        # Canary tests against DVWA/VAmPI/JuiceShop (Docker, -tags=canary)
make lint               # golangci-lint run
make fmt                # Format code
make tidy               # go mod tidy
```

Run a single test:
```bash
go test -v -run TestFunctionName ./pkg/path/to/package/...
```

Run a single test file with build tags:
```bash
go test -v -tags=e2e -run TestName ./test/e2e/...
```

## Architecture

### Execution Pipeline (Native Scan)

Request ingestion → Scope filtering → Executor (worker pool) → Module dispatch → Result output/storage

This is the **native scan** pipeline — deterministic, Go-based scanning with no AI involvement. The **Executor** (`pkg/core/executor.go`) is the central orchestrator. It receives `HttpRequestResponse` items, distributes them to registered modules via a concurrent worker pool, and collects `ResultEvent` findings. It supports pre/post hooks (`HookRunner`), scope matching, and per-host rate limiting.

### Module System

All scanner logic lives in **modules** registered in the **Registry** (`pkg/modules/registry.go`); the wiring is in `pkg/modules/default_registry_active.go` / `default_registry_passive.go` (currently 168 active + 98 passive registrations = 266 modules). Two types:

- **ActiveModule** (`pkg/modules/active.go`): Sends modified requests to detect vulnerabilities. Methods: `ScanPerInsertionPoint`, `ScanPerRequest`, `ScanPerHost`. Each module declares which `ScanScope` and `InsertionPointType` it handles.
- **PassiveModule** (`pkg/modules/passive.go`): Analyzes existing request/response pairs without sending new traffic. Optional `Flusher` interface for end-of-scan finalization.

Both share the base `Module` interface (ID, Name, Severity, Confidence, Tags, CanProcess, ScanScopes). Modules are tagged with classification labels (e.g., `spring`, `xss`, `light`) and can be filtered with `--module-tag` CLI flag or `?tag=` API parameter.

Module helper code lives in `pkg/modules/modkit/` (shared constants, default implementations) and `pkg/modules/infra/` (block detection, request filtering, response transfer).

### Ignored Directories

- **`platform/`** — Contains external tooling only. Do not read or modify files in this directory, except for `platform/vigolium-workbench/` which is the Next.js UI frontend — only go into it when making changes to the UI.

### Key Packages

- **`pkg/core/`** — Executor, worker pool, rate limiter, network utilities, scan statistics
- **`pkg/modules/`** — Module interfaces, registry, all active/passive scanner modules
- **`pkg/deparos/`** — Spider & discovery engine: crawling (`discovery/`), JS analysis (`jsscan/`), fingerprinting (`fingerprint/`), Wayback integration (`wayback/`), scope enforcement (`scope/`), WAF detection (`waf/`), storage (`storage/`)
- **`pkg/agent/`** — Agentic scan engine: prompt templates, context enrichment (`autopilot_context.go`), the autopilot pipeline runner (`autopilot_pipeline.go`) and swarm runner (`swarm.go`), output parsing (findings/HTTP records/attack plans/triage results/source analysis), and database ingestion. All AI dispatch goes through the in-process olium engine via `olium_adapter.go` — there are no subprocess SDK backends. Powers the agentic scan modes (autopilot, swarm) and the query mode.
- **`pkg/olium/`** — In-process Go agent runtime: provider drivers (`provider/` for openai-codex-oauth, anthropic-api-key, anthropic-oauth, openai-api-key, anthropic-cli), turn-based engine (`engine/`), tool registry and built-ins (`tool/`), skills support (`skill/`), the autopilot agentic loop (`autopilot/`), TUI (`tui/`), and headless one-shot helper (`headless.go`). Used by every agent mode and exposed directly via `vigolium agent olium` (alias `vigolium olium` / `ol`).
- **`pkg/audit/`** — Vigolium Audit harness driver: parser for the audit's on-disk output (`pkg/audit/parser.go`, `constants.go`), embedded binary management (`pkg/audit/bin/`), per-platform cost/stream support (`claudecost/`, `codexcost/`, `stream/`). Drives the `vigolium agent audit` foreground mode.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vigolium/vigolium](https://github.com/vigolium/vigolium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
