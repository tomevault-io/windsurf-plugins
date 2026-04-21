---
trigger: always_on
description: Kilroy is a local-first Go CLI for running software-factory pipelines in a Git repository. There is a skill to convert English requirements into DOT graphs. Then it validates graph semantics and executes stages with checkpoint commits and a run history backed by `cxdb`. Foundational specs that are in docs/strongdm/attractor.
---

# Repository Guidelines

## What Kilroy Is
Kilroy is a local-first Go CLI for running software-factory pipelines in a Git repository. There is a skill to convert English requirements into DOT graphs. Then it validates graph semantics and executes stages with checkpoint commits and a run history backed by `cxdb`. Foundational specs that are in docs/strongdm/attractor.

Use Kilroy in this order: build the binary, generate or write a graph, validate it, then run it with a config file. Typical flow: `go build -o ./kilroy ./cmd/kilroy`, `./kilroy attractor ingest -o pipeline.dot "<requirements>"`, `./kilroy attractor validate --graph pipeline.dot`, then `./kilroy attractor run --graph pipeline.dot --config run.yaml`.

## What you're doing here - **the Prime Directive**.
If you can see this message, then you are not here to use Kilroy - **YOU ARE HERE TO IMPROVE KILROY**. If Kilroy fails to build a project:
- Don't fix the project
- Don't fix the dotfile
- Don't fix the system so it works for *this* project
Use the knowledge you've gained from the failure to make the system more robust for *every* project. Your changes should work for every project, every language, every system.
Of course, **specific user instructions may override this**, or any other section.

# Think like a user
Think about a blank slate agent that is trying to properly create a dotfile using the dotfile skill and then run it with the attractor. What mistakes would it make? What ergonomics would steer it away? How can you make that robust for every possible project the attractor could work on, not just this one? **How can you do that without asking it to know the impossible, like how hard a problem is or how long something might take?**

## Canonical Specs

These three specs are the true north for system design. If you are making a change that relates to one of their areas, you must consult the relevant spec first to see what the idiomatic solution is.

- **Unified LLM Spec** (`docs/strongdm/attractor/unified-llm-spec.md`): Provider-agnostic LLM client — a single `Client` interface across LLM endpoints with unified types, retry/backoff, streaming, and tool calling. Key implementation: `internal/llm/` (client, types, errors, retry, generate, streaming) and `internal/llm/providers/` (per-provider adapters).

- **Attractor Spec** (`docs/strongdm/attractor/attractor-spec.md`): DOT-graph pipeline runner — parses Graphviz DOT into a directed graph of stages (LLM tasks, tool gates, human gates, parallel fan-out/fan-in) and executes them with conditional edge routing, checkpoint/resume, model stylesheets, and retry policies. Key implementation: `internal/attractor/dot/` (parser, lexer), `internal/attractor/engine/` (execution, handlers, parallel, resume, failure policy), `internal/attractor/validate/` (graph linting), `internal/attractor/style/` (CSS-like model stylesheet), `internal/attractor/runtime/` (checkpoint, context, status).

- **Coding Agent Loop Spec** (`docs/strongdm/attractor/coding-agent-loop-spec.md`): Turn-based agentic loop — pairs an LLM with developer tools (file edit, shell, search, glob, grep) through repeated LLM-call → tool-execution cycles with context truncation, subagent spawning, and event-driven observation. Key implementation: `internal/agent/` (session.go for the loop, tool_registry.go for tool dispatch, profile.go for provider-specific toolsets, env_local.go for filesystem/shell execution, events.go for the event bus).

## Project Structure & Module Organization
- `cmd/kilroy/`: CLI entrypoint and subcommands for `attractor` commands (`run`, `resume`, `status`, `stop`, `validate`, `ingest`).
- `internal/attractor/`: core engine/runtime, graph validation, config loading, and model metadata handling.
- `internal/agent/`, `internal/cxdb/`, `internal/llmclient/`: coding-agent loop, CXDB integration, and provider client/env wiring.
- `scripts/`: operational helpers (`e2e.sh`, `e2e-guardrail-matrix.sh`, `start-cxdb.sh`, `run_benchmarks.sh`).
- `demo/`, `docs/`, `skills/`: sample graphs, architecture/spec references, and ingestion skills.

## Skill Symlink Layout
- Canonical skill content lives under `skills/<name>/`.
- `.claude/skills/<name>` must be a symlink to `../../skills/<name>`.
- `.agents/skills/<name>` must be a symlink to `../../.claude/skills/<name>`.
- When adding/removing/renaming repo skills, update both symlink directories in the same change.

## Build, Test, and Development Commands
- `go build -o ./kilroy ./cmd/kilroy`: build the local CLI binary.
- `go test ./...`: run the full Go test suite.
- `./scripts/e2e.sh`: smoke check (tests, build, and graph validation).
- `./scripts/e2e-guardrail-matrix.sh`: run targeted engine guardrail regression tests.
- `./kilroy attractor validate --graph <file.dot>`: validate graph structure/semantics before execution.

## Pre-Commit CI Checklist

Run all of these before every commit. They mirror the exact CI steps in `.github/workflows/ci.yml` and **all must pass** before pushing:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danshapiro/kilroy](https://github.com/danshapiro/kilroy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
