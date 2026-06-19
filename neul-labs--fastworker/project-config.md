---
trigger: always_on
description: This document helps Claude Code and other AI assistants understand the FastWorker codebase structure, architecture, and development practices.
---

# CLAUDE.md - FastWorker Project Guide

This document helps Claude Code and other AI assistants understand the FastWorker codebase structure, architecture, and development practices.

## Project Overview

**FastWorker** is a brokerless task queue for Python applications with automatic worker discovery, priority handling, and built-in management GUI. It eliminates the need for external message brokers like Redis or RabbitMQ by using a control plane architecture with NNG (nanomsg-next-generation) for messaging. Formal state machines manage task, worker, subworker, and client lifecycles with atomic transitions for safe concurrency.

**Target Use Case**: Moderate-scale Python applications (1K-10K tasks/min)
**Language**: Python 3.12+
**Key Dependencies**: pynng, pydantic
**Frontend**: Vue.js 3, TailwindCSS (for management GUI)
**License**: MIT

## Architecture

### Control Plane Architecture

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │ TCP (via control plane)
       │
┌──────▼──────────────┐
│  Control Plane      │ (Coordinator + Task Processor)
│  - Task distribution│
│  - Result caching   │
│  - Worker registry  │
└──────┬──────────────┘
       │
   ┌───┴───┬────────┐
   │       │        │
┌──▼───┐ ┌▼────┐ ┌─▼────┐
│Sub-  │ │Sub- │ │Sub-  │
│worker│ │worker│ │worker│
└──────┘ └─────┘ └──────┘
```

**Key Components**:
1. **Control Plane Worker**: Central coordinator that manages task distribution and also processes tasks
2. **Subworkers**: Additional workers that register with control plane for load distribution
3. **Clients**: Connect to control plane for task submission and result retrieval
4. **Discovery Service**: Enables workers to find the control plane automatically
5. **Management GUI**: Built-in web dashboard for monitoring (Vue.js + TailwindCSS)

## Directory Structure

```
fastworker/
├── fastworker/              # Main package
│   ├── __init__.py         # Package exports (task, Client, etc.)
│   ├── cli.py              # CLI commands (control-plane, subworker, submit, etc.)
│   ├── main.py             # Main entry point
│   │
│   ├── clients/            # Client implementations
│   │   ├── client.py       # Main client for task submission
│   │   └── discovery.py    # Discovery service client
│   │
│   ├── workers/            # Worker implementations
│   │   ├── control_plane.py # Control plane worker
│   │   └── subworker.py    # Subworker implementation
│   │
│   ├── tasks/              # Task management
│   │   ├── models.py       # Task models (Task, TaskResult, TaskPriority)
│   │   ├── registry.py     # Task registry and decorator
│   │   ├── state.py        # TaskStateMachine (9-state lifecycle)
│   │   └── serializer.py   # Task serialization
│   │
│   ├── utils/              # Utilities
│   │   ├── state_machine.py # Generic async StateMachine[S]
│   │   └── event_bus.py    # asyncio.Queue-based pub/sub EventBus
│   │
│   ├── patterns/           # NNG communication patterns
│   │   └── nng_patterns.py # ReqRep, Bus, Pair, SurveyorRespondent
│   │
│   ├── telemetry/          # Optional OpenTelemetry integration
│   │   ├── tracer.py       # Distributed tracing
│   │   └── metrics.py      # Metrics collection
│   │
│   ├── gui/                # Management GUI
│   │   ├── __init__.py     # Package exports
│   │   ├── server.py       # HTTP server with REST API
│   │   ├── static/         # Pre-built Vue.js frontend
│   │   └── frontend/       # Vue.js source code
│   │       ├── src/        # Vue components
│   │       ├── package.json
│   │       └── build.sh    # Build script
│   │
│   └── examples/           # Example code
│       ├── tasks.py        # Example task definitions
│       ├── fastapi_example.py  # FastAPI integration example
│       └── callback_example.py # Task completion callbacks
│
├── tests/                  # Test suite
├── docs/                   # Documentation
│   ├── index.md           # Documentation index
│   ├── api.md             # API reference
│   ├── gui.md             # Management GUI guide
│   ├── limitations.md     # Scope and limitations
│   ├── fastapi.md         # FastAPI integration guide
│   └── telemetry.md       # OpenTelemetry guide
│
├── pyproject.toml         # uv/PEP 621 configuration
├── README.md              # User-facing documentation
└── CONTRIBUTING.md        # Contribution guidelines
```

## Key Files and Their Purposes

### Core Files

- **`fastworker/__init__.py`**: Package exports - defines public API (`task`, `Client`, `TaskPriority`)
- **`fastworker/cli.py`**: CLI commands implementation (typer-based)
- **`fastworker/tasks/registry.py`**: Task decorator and registration system
- **`fastworker/tasks/models.py`**: Core data models (Task, TaskResult, TaskPriority, TaskStatus)
- **`fastworker/tasks/state.py`**: TaskStateMachine with 9 states and atomic transitions
- **`fastworker/workers/state.py`**: WorkerStateMachine with 6 states (INIT→RUNNING→STOPPED)
- **`fastworker/utils/state_machine.py`**: Generic async StateMachine base class with asyncio.Lock
- **`fastworker/utils/event_bus.py`**: EventBus for publishing state transitions to GUI/telemetry

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neul-labs/fastworker](https://github.com/neul-labs/fastworker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
