---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**workflow-builder** ("LOOM") is a pip-installable, **library-first** durable execution SDK for AI-powered workflows. The primary deliverable is a **Workflow Coding Agent**: an LLM-powered agent that _authors_ workflow code — mixing third-party SDK calls, workflow constructs (`@workflow`, `@step`, `ctx.*`), and raw Python — so that users describe what they want and receive a ready-to-run workflow.

Generated workflows are execution-portable: they run embedded (SQLite store, no infra), in a user-supplied sandbox, or against an external durable backend (Temporal, DBOS, Restate). The SDK never forces a specific runtime.

The SDK's core engine design is **deterministic re-entry**: workflow bodies can be safely re-executed after crashes/deploys because every side effect is journaled and served from the journal on replay. This is analogous to Temporal's event-sourced execution model.

## Commands

```bash
# Install with dev dependencies
pip install -e ".[dev]"

# Install with FastAPI webhook support
pip install -e ".[dev,api]"

# Run all tests
pytest

# Run a single test file or test
pytest tests/test_runtime.py
pytest tests/test_runtime.py::test_basic_workflow

# Lint
ruff check src tests

# Type checking
mypy
```

## Architecture

### Core Execution Loop

The runtime (`runtime/engine.py`) drives execution:
1. Load `ExecutionRecord` and `Journal` from the store
2. Re-enter the workflow body; the journal short-circuits already-completed work
3. If body completes → `COMPLETED`; if raises `Suspend` → park with wake time or event name; if raises exception → `FAILED`

**Critical invariant:** Every durable operation must go through the `Context` API so results are journaled. Calling external services directly inside a workflow body will cause double-execution on replay.

### Layer Responsibilities

| Layer | Path | Purpose |
|-------|------|---------|
| **Runtime** | `runtime/engine.py` | Re-entry loop, lifecycle (run/resume/retry/replay/cancel), scheduler tick |
| **Context** | `runtime/context.py` | The only legal API from workflow code to the outside world (`step`, `sleep`, `wait_for_event`, `call_agent`, `spawn`, `gather`) |
| **Journal** | `runtime/journal.py` | Per-run log of durable operations; provides deterministic replay |
| **Workflow** | `runtime/workflow.py` | `WorkflowDefinition` wrapper + `@workflow` decorator |
| **Steps** | `steps/definition.py` | `@step` decorator — wraps async functions with retry, timeout, fallback |
| **State** | `state/` | Pluggable persistence: `ExecutionStore` protocol, `MemoryStore`, `SQLiteStore`, `MongoStore`, `PostgresStore` |
| **Agents** | `agents/` | `ModelProvider` protocol, `Tool` abstraction (steps/workflows-as-tools), guardrails, memory |
| **Triggers** | `triggers/` | Entry points: `Webhook`, `Schedule`, `Manual`, `Poll`, `Event`, `Chat`, `Email`, `SubWorkflow` |
| **Observability** | `observability/tracing.py` | `Tracer` protocol + `NoopTracer`; plug in OTel/Datadog/Honeycomb |

### Suspension Model

Workflows park themselves by raising `Suspend(wake_at=datetime)` or `Suspend(awaiting_event="name")`. The engine persists the suspension, and `runtime.tick()` / `runtime.resume(run_id)` re-enters the workflow at the next opportunity. This is how `ctx.sleep()` and `ctx.wait_for_event()` work internally.

### Public API Surface

`src/workflow_builder/__init__.py` re-exports ~10 symbols that form the user-facing API:
`Context`, `ExecutionResult`, `ExecutionStatus`, `Failure`, `OnError`, `Retry`, `Usage`, `Runtime`, `StepContext`, `step`, `workflow`.

All internal modules (~200+ classes) are implementation details.

### Workflow Coding Agent

The agent is the primary user-facing feature. It takes a natural-language description and produces a valid, runnable workflow file. Key design constraints for the agent and the code it generates:

- **Code style:** Generated workflows use `@workflow` + `@step` + `ctx.*` for all durable operations. Raw Python and third-party SDK calls belong inside `@step` bodies, never directly in the workflow body.
- **Tools available to the coding agent:** Any `@step` or `WorkflowDefinition` can be surfaced as a `Tool` via `tool_from_step()` / `tool_from_workflow()` / `coerce_tool()` in `agents/tools.py`. The agent's toolset is therefore the SDK itself — it can call steps and sub-workflows as tools to introspect capabilities.
- **Schema derivation:** Tool schemas are derived from function signatures + docstring `Args:` sections (`agents/tools.py::build_parameter_schema`). Keep docstrings accurate — they are the source of truth for the model.
- **Execution target:** Generated code must be runnable with just `pip install workflow-builder` and `MemoryStore` (no external infra). Sandbox or cloud execution is a deployment detail, not a code change.
- **Agent persistence:** Coding sessions are durable artifacts. The authoring session (spec, decision log, diagnostics) is itself a workflow run that can be resumed — use `session`/`persistent` agent classes, not ephemeral.

### Extension Points

- **Custom persistence:** Implement `state/base.py::ExecutionStore`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pipeshub-ai/loom](https://github.com/pipeshub-ai/loom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
