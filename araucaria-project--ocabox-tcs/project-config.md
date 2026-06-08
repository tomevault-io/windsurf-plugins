---
trigger: always_on
description: This is the **OCM Telescope Control Services (ocabox-tcs)** repository - a universal Python service framework for telescope control systems built around NATS messaging. The framework is execution-method agnostic and supports multiple deployment scenarios.
---

# GitHub Copilot Instructions for ocabox-tcs

## Repository Overview

This is the **OCM Telescope Control Services (ocabox-tcs)** repository - a universal Python service framework for telescope control systems built around NATS messaging. The framework is execution-method agnostic and supports multiple deployment scenarios.

## Core Architecture

### Universal Service Framework

- **Service Independence**: Services work across manual/process/asyncio/systemd execution methods
- **Service Types**: Permanent, blocking permanent, and single-shot services
- **Decorator-Based**: Modern Python decorators (`@service`, `@config`) for clean registration
- **Filename-Based Discovery**: Service type automatically derived from filename (e.g., `hello_world.py` → `hello_world`)
- **Optional Configuration**: Config classes are optional, services can use base config
- **Distributed Management**: NATS-based service discovery, management, and monitoring
- **Flexible Deployment**: Services in local or external packages

### Key Directories

```
src/
├── ocabox_tcs/
│   ├── base_service.py          # Base classes and decorators
│   ├── service_controller.py    # Single service lifecycle control
│   ├── process_context.py       # Shared resources (Singleton)
│   ├── launchers/               # Execution launchers
│   │   ├── process.py           # Process-based launcher
│   │   ├── asyncio.py           # Asyncio-based launcher
│   │   └── base_launcher.py     # Base classes
│   ├── monitoring/              # Monitoring system
│   │   ├── status.py            # Status enum and StatusReport
│   │   ├── monitored_object.py  # Base monitoring classes
│   │   └── monitored_object_nats.py  # NATS-enabled monitoring
│   └── services/                # Available services
│       ├── hello_world.py       # Canonical template service
│       └── examples/            # Tutorial examples (01-05)
└── tcsctl/                      # CLI tool (separate package)
    ├── app.py                   # Main CLI entry point
    ├── client.py                # Service control client
    └── display.py               # Rich terminal output
```

## Development Guidelines

### Code Style

- **Python Version**: 3.12+ required
- **Formatter**: Black with 100 character line length (`poetry run black .`)
- **Linter**: Ruff for catching real issues (gentle rules, non-annoying)
- **Import Style**: Absolute imports (e.g., `from ocabox_tcs.base_service import service`)
- **Comments**: Only add comments if they match existing style or explain complex logic
- **Naming**: Use descriptive names, follow existing patterns

### Framework Naming Conventions

**Framework-Provided Attributes** (prefixed with `svc_` to avoid collisions):
- `self.svc_logger`: Logger instance configured for the service
- `self.svc_config`: Service configuration object
- `self.controller`: Reference to ServiceController
- `self.monitor`: Monitoring object for status/health reporting
- `self.is_running`: Boolean property indicating if service is running

**Users are FREE to use**:
- `self.logger`, `self.config`, etc. for their own purposes
- Any non-prefixed attribute names

### Service Creation Patterns

#### Non-Blocking Permanent Service (for async tasks/workers)

```python
from ocabox_tcs.base_service import service, BasePermanentService

@service
class MyService(BasePermanentService):
    async def start_service(self):
        # Spawn background tasks/workers
        self.task = asyncio.create_task(self.worker())

    async def stop_service(self):
        # Clean up background tasks
        self.task.cancel()
        await self.task
```

#### Blocking Service with Main Loop (most common pattern)

```python
from ocabox_tcs.base_service import service, BaseBlockingPermanentService

@service
class WorkerService(BaseBlockingPermanentService):
    async def on_start(self):
        """Optional: Called before run_service starts - for setup."""
        pass

    async def run_service(self):
        """REQUIRED: Main service loop - runs in managed task."""
        while self.is_running:
            # Main work loop
            await asyncio.sleep(1)

    async def on_stop(self):
        """Optional: Called after run_service stops - for cleanup."""
        pass
```

**CRITICAL**: `BaseBlockingPermanentService` enforces its API at import time:
- ✅ **Override**: `run_service()` - your main loop (required)
- ✅ **Override**: `on_start()`, `on_stop()` - optional hooks
- ❌ **DO NOT override**: `start_service()`, `stop_service()` - these manage task lifecycle
- Violating this raises `TypeError` at class definition time

#### Single-Shot Service

```python
from ocabox_tcs.base_service import service, BaseSingleShotService

@service
class DataImporterService(BaseSingleShotService):
    async def execute(self):
        # One-time task
        pass
```

### Status Management

**The framework automatically manages status transitions!** Manual calls are usually unnecessary.

**Automatic Status Lifecycle**:
1. `STARTUP` - Set by controller during initialization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [araucaria-project/ocabox-tcs](https://github.com/araucaria-project/ocabox-tcs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-07 -->
