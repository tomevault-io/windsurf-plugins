---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Test Commands

```bash
# Build
make build              # Build to bin/osmedeus
make build-all          # Cross-platform builds (linux, darwin, windows)

# Test
make test-unit          # Fast unit tests (no external dependencies)
make test-integration   # Integration tests (requires Docker)
make test-e2e           # E2E CLI tests (requires binary build)
make test-e2e-ssh       # SSH E2E tests (module & step level SSH runner)
make test-e2e-api       # API E2E tests (all endpoints with Redis + seeded DB)
make test-e2e-cloud     # Cloud E2E tests (cloud CLI commands)
make test-sudo          # Sudo-aware E2E tests (requires interactive sudo prompt)
make test-cloud         # Cloud integration tests (internal cloud package)
make test-distributed   # Distributed run e2e tests (requires Docker for Redis)
make test-docker        # Docker runner tests
make test-ssh           # SSH runner unit tests (starts test SSH container)
make test-canary-all    # Canary tests: real scans in Docker (30-60min)
make test-canary-repo   # Canary: SAST scan on juice-shop (~25min)
make test-canary-domain # Canary: domain recon on hackerone.com (~20min)
make test-canary-ip     # Canary: CIDR scan on IP list (~25min)
make test-canary-general # Canary: domain-list-recon on hackerone.com subdomains (~40min)
go test -v ./internal/functions/...  # Run tests for specific package
go test -v -run TestName ./...       # Run single test by name

# Development
make fmt                # Format code
make lint               # Run golangci-lint
make tidy               # go mod tidy
make run                # Build and run

# Installation
make install            # Install to $GOBIN (or $GOPATH/bin)
make swagger            # Generate Swagger documentation

# Docker Toolbox
make docker-toolbox       # Build toolbox image (all tools pre-installed)
make docker-toolbox-run   # Start toolbox container
make docker-toolbox-shell # Enter toolbox container shell

# Docker Canary (real-world scan testing)
make canary-up            # Build & start canary container
make canary-down          # Stop & cleanup canary container

# UI
make update-ui          # Update embedded UI from dashboard build
```

## Architecture Overview

Osmedeus is a workflow engine for security automation. It executes YAML-defined workflows with support for multiple execution environments.

### Layered Architecture

```
CLI/API (pkg/cli, pkg/server)
         ↓
Executor (internal/executor) - coordinates workflow execution
         ↓
StepDispatcher - routes to: BashExecutor, FunctionExecutor, ForeachExecutor, ParallelExecutor, RemoteBashExecutor, HTTPExecutor, LLMExecutor, AgentExecutor, ACPExecutor
         ↓
Runner (internal/runner) - executes commands via: HostRunner, DockerRunner, SSHRunner
```

### Core Packages

| Package | Purpose |
|---------|---------|
| `internal/core` | Type definitions: Workflow, Step, Trigger, RunnerConfig, ExecutionContext |
| `internal/parser` | YAML parsing, validation, and caching (Loader) |
| `internal/executor` | Workflow execution engine with step dispatching |
| `internal/runner` | Execution environments implementing Runner interface |
| `internal/template` | `{{Variable}}` interpolation engine |
| `internal/functions` | Utility functions via Goja JavaScript VM |
| `internal/scheduler` | Cron, event, and file-watch triggers (fsnotify-based) |
| `internal/database` | SQLite/PostgreSQL via Bun ORM |
| `pkg/cli` | Cobra CLI commands |
| `pkg/server` | Fiber REST API |
| `internal/snapshot` | Workspace export/import as compressed ZIP archives |
| `internal/installer` | Binary installation (direct-fetch and Nix modes) |
| `internal/state` | Run state export for debugging and sharing |
| `internal/updater` | Self-update functionality via GitHub releases |
| `internal/cloud` | Cloud infrastructure provisioning (DigitalOcean, AWS, GCP, Linode, Azure) |

### Key Types

```go
WorkflowKind: "module" | "flow"  // module = single unit, flow = orchestrates modules
StepType: "bash" | "function" | "parallel-steps" | "foreach" | "remote-bash" | "http" | "llm" | "agent" | "agent-acp"
RunnerType: "host" | "docker" | "ssh"
TriggerType: "cron" | "event" | "watch" | "manual"
```

### Decision Routing

Steps support conditional branching via `decision` field with switch/case syntax:
```yaml
decision:
  switch: "{{variable}}"
  cases:
    "value1": { goto: step-a }
    "value2": { goto: step-b }
  default: { goto: fallback }
```
Use `goto: _end` to terminate workflow.

### Workflow Execution Flow

1. CLI parses args ▷ loads config from `~/osmedeus-base/osm-settings.yaml`
2. Parser loads YAML workflow, validates, caches in Loader
3. Executor initializes context with built-in variables (`{{Target}}`, `{{Output}}`, etc.)
4. StepDispatcher routes each step to appropriate executor
5. Runner executes commands, captures output
6. Exports propagate to subsequent steps

### Template System

- `{{Variable}}` - standard template variables (Target, Output, threads, etc.)
- `[[variable]]` - foreach loop variables (to avoid conflicts)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [j3ssie/osmedeus](https://github.com/j3ssie/osmedeus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
