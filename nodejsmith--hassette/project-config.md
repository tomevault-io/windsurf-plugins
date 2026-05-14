---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hassette is an async-first Python framework for building Home Assistant automations. It emphasizes type safety (Pydantic models), dependency injection (FastAPI-style), and async/await patterns. Python 3.11+ required.

## Common Commands

```bash
# Install dependencies
uv sync

# Run tests locally (preferred for development)
uv run nox -s dev

# Run tests via nox (CI — tests across Python 3.11, 3.12, 3.13)
uv run nox -s tests

# Run tests with coverage
uv run nox -s tests_with_coverage

# Run a single test file
uv run pytest tests/integration/test_api.py

# Run a specific test
uv run pytest tests/integration/test_api.py::test_function_name -v

# Type checking
uv run pyright

# Serve documentation locally
uv run mkdocs serve
```

## Architecture

### Core Components

**Hassette** (`src/hassette/core/core.py`) - Main coordinator that connects to Home Assistant via WebSocket, manages app lifecycle, and coordinates all services.

**App** (`src/hassette/app/app.py`) - Base class for user automations. Generic over `AppConfig` type. Each app gets its own Bus, Scheduler, Api, and StateManager. Lifecycle hooks: `on_initialize`, `on_ready`, `on_shutdown`.

**Bus** (`src/hassette/bus/`) - Event pub/sub with filtering. Methods: `on_state_change`, `on_attribute_change`, `on_call_service`, `on`. Supports glob patterns, predicates, conditions, debounce, throttle.

**Scheduler** (`src/hassette/scheduler/`) - Task scheduling via trigger objects. Primary entry: `schedule(func, trigger)`. Convenience methods: `run_in()`, `run_once()`, `run_every()`, `run_daily()`, `run_cron()`. Trigger types: `After`, `Once`, `Every`, `Daily`, `Cron` (all in `hassette.scheduler.triggers`). Custom triggers implement `TriggerProtocol`. Supports job groups (`group=`, `cancel_group()`, `list_jobs(group=)`) and jitter (`jitter=`).

**Api** (`src/hassette/api/`) - Home Assistant REST/WebSocket interface. Async methods: `get_state()`, `get_states()`, `call_service()`, `set_state()`, `fire_event()`.

**StateManager** (`src/hassette/state_manager/`) - State access and caching with type conversion. Supports domain access (`self.states.light`), generic access (`self.states[CustomState]`), and direct entity lookup (`self.states.get("light.kitchen")`).

### Event Handling Modules

Located in `src/hassette/event_handling/`:
- `predicates.py` (aliased as `P`) - Event matching predicates
- `conditions.py` (aliased as `C`) - Value comparison conditions
- `accessors.py` (aliased as `A`) - Field extraction helpers
- `dependencies.py` (aliased as `D`) - Dependency injection

### Type Conversion Registries

- `STATE_REGISTRY` - Maps Home Assistant entity types to Python model classes
- `TYPE_REGISTRY` - Maps scalar types for field conversion

### Resource Hierarchy

`Resource` (`src/hassette/resources/base.py`) is the base class for app components (Bus, Scheduler, Api, StateManager). `Service` extends it for background services. Both have lifecycle hooks and child resource tracking with priority-based initialization/shutdown.

Services declare a `restart_spec` class attribute (`RestartSpec`) that controls supervision behavior: restart type (`PERMANENT`, `TRANSIENT`, or `TEMPORARY`), sliding-window budget (intensity + period), backoff parameters, and error routing (fatal vs. non-retryable error names). The `ServiceWatcher` reads this spec when a service fails.

## App Pattern

```python
class MyConfig(AppConfig):
    model_config = SettingsConfigDict(env_prefix="my_")
    setting_name: str = "default"

class MyApp(App[MyConfig]):
    async def on_initialize(self):
        self.bus.on_state_change("light.kitchen", handler=self.on_light_change)
        self.scheduler.run_in(self.my_task, 5)

    async def on_light_change(self, event: RawStateChangeEvent):
        pass
```

## Bug Investigation Workflow (TDD)

When investigating a crash or regression, follow this sequence before writing any fix:

1. **Reproduce first** — confirm the bug is real and understand what triggers it (logs, crash output, minimal repro)
2. **Write a failing test** — write a test that captures the exact failure mode; run it and confirm it fails (RED)
3. **Fix the code** — write the minimal change that makes the test pass (GREEN)
4. **Verify** — run the full test file to ensure no regressions; check the new test passes and existing tests still pass

This discipline matters most for startup races, timing bugs, and subtle state issues — categories where "it seemed to work" is not trustworthy evidence.

### Regression test patterns for this project

**Startup races** — use `asyncio.Event` as a gate to simulate a dependency not yet ready:
```python
gate = asyncio.Event()
mock_service.wait_for_ready = AsyncMock(side_effect=lambda _: gate.wait())
task = asyncio.create_task(executor.register_listener(...))
await asyncio.sleep(0)         # let the task run until it blocks on gate
assert not task.done()         # confirms the gate is actually blocking it
gate.set()
await task
assert result > 0              # confirms registration succeeded after unblocking
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NodeJSmith/hassette](https://github.com/NodeJSmith/hassette) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
