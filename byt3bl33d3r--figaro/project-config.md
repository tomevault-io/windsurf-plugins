---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Figaro is a NATS-based orchestration system for managing Claude agent workers running in containerized desktop environments. Workers execute browser automation tasks via the claude-agent-sdk, with live desktop streaming through Apache Guacamole (guacd + guacamole-common-js). A supervisor agent handles task optimization and delegation. A channel-agnostic gateway service routes messages to/from external channels (Telegram, etc.) for human-in-the-loop interactions.

All services communicate via NATS (pub/sub + JetStream for durable task events). The UI connects to NATS via WebSocket (nats.ws) for both real-time events and mutations (request/reply). HTTP endpoints are minimal: `GET /api/config` (NATS URL discovery), `GET /api/guacamole/token` (encrypted connection tokens), and `WS /guacamole/webSocket` (Guacamole WebSocket tunnel via guapy).

## Long-Running Task Design

**This system is built for tasks that run for minutes to hours.** Browser automation with Claude agents is inherently long-running — a single task may involve navigating dozens of pages, filling forms, waiting for page loads, and interacting with complex web applications. All timeouts, subscriptions, and communication patterns must account for this.

Key principles:
- **Never use fixed wall-clock timeouts for task execution.** Use inactivity-based timeouts that reset on worker progress. A task actively streaming messages is healthy regardless of how long it's been running.
- **NATS request/reply timeouts are for API calls, not task lifecycles.** The `request()` timeout (default 10s, UI 30s) covers orchestrator round-trips, NOT how long a task takes. Task progress is tracked via JetStream subscriptions, not request/reply.
- **JetStream provides durable delivery.** Task events (messages, completion, errors) go through JetStream so they survive reconnections. Core NATS is only for ephemeral operations (registration, heartbeats, API calls).
- **Supervisor delegation is blocking but progress-aware.** `waitForDelegation()` in `supervisor/tools.ts` subscribes to `figaro.task.{id}.message` and resets its inactivity timer on every worker message. The `DELEGATION_INACTIVITY_TIMEOUT` (600s) is a silence detector, not a task duration limit.
- **Help requests have their own timeouts.** Human-in-the-loop requests wait independently (default 300s) and don't affect the task execution timeout.
- **Use Core NATS subscriptions for waiting on task events in the worker/supervisor.** JetStream ephemeral push consumers (`js.subscribe()` in nats.js v2) are unreliable — they can fail silently or throw during setup, causing tools like `delegate_to_worker` to return immediately instead of blocking. Since JetStream publishes also deliver to Core NATS subscribers on the same subject, use `nc.subscribe()` instead (same pattern as `HelpRequestHandler`). Both are ephemeral and behave identically for `DeliverPolicy.New` use cases.

When adding new features or modifying existing ones, always ask: "What happens if this task runs for 2 hours?" If the answer involves a timeout killing it while it's still making progress, the design is wrong.

## Bug Fixing Process

When I report a bug, don't start by trying to fix it. Instead, start by writing a test that reproduces the bug. Then, have subagents try to fix the bug and prove it with a passing test.

## Code Style

- **Use f-strings for all string interpolation.** No `%s`-style formatting or `.format()` calls — always use f-strings.
- **Use high-level asyncio APIs.** Prefer `asyncio.create_task()`, `asyncio.gather()`, `asyncio.wait_for()`, etc. over low-level primitives like `loop.create_future()`, `ensure_future()`, or direct event loop access.
- **Use `pathlib` for all path operations.** No `os.path` calls — always use `pathlib.Path` for constructing, joining, and manipulating file paths.
- **Avoid nested functions.** Don't define functions inside other functions — extract them as module-level or class-level methods instead.
- **No inline imports.** All imports must be at the top of the file — never import inside functions, methods, or conditional blocks.
- **Use `Subjects` constants for all NATS subjects.** Never use raw strings like `"figaro.broadcast.foo"` — always use `Subjects.SOME_CONSTANT` or `Subjects.some_method(id)` from `figaro_nats.Subjects` (Python) or `nats/subjects.ts` (TypeScript). When adding a new NATS subject, add the constant to `figaro-nats/src/figaro_nats/subjects.py` first, then reference it everywhere.
- **Keep files under 500 lines.** If a file exceeds 500 lines, split it into smaller, focused modules.

## Build & Run Commands

### Shared NATS Library (figaro-nats/)
```bash
cd figaro-nats
uv sync --frozen
uv run pytest             # Run tests
uv run ruff check .       # Lint
uv run ruff format .      # Format
```

### Orchestrator (figaro/)
```bash
cd figaro
uv sync --frozen          # Install dependencies
uv run figaro             # Start orchestrator on port 8000
uv run pytest             # Run all tests
uv run pytest tests/test_registry.py -k "test_register"  # Single test
uv run ruff check .       # Lint

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [byt3bl33d3r/figaro](https://github.com/byt3bl33d3r/figaro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
