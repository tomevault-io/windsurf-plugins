---
trigger: always_on
description: This file provides project guidance for coding agents working in this repository.
---

# CLAUDE.md

This file provides project guidance for coding agents working in this repository.

## HIGH PRIORITY

- **IF YOU DON'T CHECK SKILLS** your task will be invalidated and we will generate rework
- **YOU CAN ONLY** finish a task if `make verify` passes at 100% (runs `fmt + lint + test + build`). No exceptions — failing any of these commands means the task is **NOT COMPLETE**
- **`make lint` has zero tolerance**. **Zero issues allowed** — any golangci-lint issue is a blocking failure
- **ALWAYS** check dependent package APIs before writing integration code or tests to avoid writing wrong code
- **NEVER** use workarounds — always use the `no-workarounds` skill for any fix/debug task + `testing-anti-patterns` for tests
- **ALWAYS** use the `no-workarounds` and `systematic-debugging` skills when fixing bugs or complex issues
- **NEVER** use web search tools to search local project code — for local code, use Grep/Glob instead
- **YOU SHOULD NEVER** add dependencies by hand in `go.mod` — always use `go get` instead

## MANDATORY REQUIREMENTS

- **MUST** run `make verify` before completing ANY subtask
- **ALWAYS USE** the `golang-pro` skill before writing any Go code
- **ALWAYS USE** the `systematic-debugging` + `no-workarounds` skills before fixing any bug
- **ALWAYS USE** the `testing-anti-patterns` skill before writing or modifying tests
- **ALWAYS USE** the `cy-final-verify` skill before claiming any task is done
- **Skipping any verification check will result in IMMEDIATE TASK REJECTION**

## Project Overview

Compozy is a Go module and CLI that drives the full lifecycle of AI-assisted development. It covers product ideation (PRD creation), technical specification, task breakdown with codebase-informed enrichment, and automated execution of each task via AI coding agents (Claude Code, Codex, Droid, Cursor). It also handles PR review remediation workflows.

## Package Layout

| Path                          | Responsibility                                                             |
| ----------------------------- | -------------------------------------------------------------------------- |
| `cmd/compozy`                 | Standalone CLI entry point                                                 |
| `compozy.go`                  | Public Go API — `NewCommand()` embeds the CLI as a Cobra subcommand        |
| `agents`                      | Bundled council agent definitions (embedded via `embed.go`)                |
| `internal/cli`                | Cobra flags, interactive form collection, CLI glue                         |
| `internal/core`               | Internal facade for reusable preparation and execution                     |
| `internal/core/agent`         | IDE command validation and process command construction                    |
| `internal/core/agents`        | Reusable agent discovery, validation, MCP merge, and catalog               |
| `internal/core/extension`     | Extension manifest, discovery, lifecycle, Host API, review provider bridge |
| `internal/core/kernel`        | Typed command dispatcher, handler registry, and legacy adapters            |
| `internal/core/model`         | Shared runtime data structures                                             |
| `internal/core/modelprovider` | Model provider alias discovery and resolution                              |
| `internal/core/plan`          | Input discovery, filtering, grouping, and batch prep                       |
| `internal/core/prompt`        | Thin prompt builders that emit runtime context and skill names             |
| `internal/core/run`           | Execution pipeline, logging, shutdown, and Bubble Tea UI                   |
| `internal/core/run/exec`      | Ad-hoc single-prompt execution pipeline with extension hooks               |
| `internal/core/run/executor`  | Workflow execution engine, event streaming, and lifecycle                  |
| `internal/core/run/journal`   | Durable append-before-publish event journal for run artifacts              |
| `internal/core/provider`      | Review provider abstraction, overlay registry, and CodeRabbit adapter      |
| `pkg/compozy/events`          | Public event envelope, kind constants, and documented payload API          |
| `pkg/compozy/runs`            | Public run reader library for list/open/replay/tail/watch flows            |
| `sdk/extension`               | Go SDK for extension authors (hooks, Host API, review providers)           |
| `sdk/extension-sdk-ts`        | TypeScript SDK for extension authors                                       |
| `skills`                      | Bundled installable skills (creation + execution workflows)                |
| `.compozy/tasks`              | Default workflow artifact root (PRD, TechSpec, ADR, reviews)               |
| `internal/version`            | Build metadata                                                             |

## Build & Development Commands

```bash
# Full verification pipeline (BLOCKING GATE for any change)
make verify              # Serial: fmt -> lint -> test -> build

# Individual steps
make fmt                 # Format with gofmt
make lint                # Strict golangci-lint (zero issues tolerance)
make test                # Run tests with -race flag

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [compozy/compozy](https://github.com/compozy/compozy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
