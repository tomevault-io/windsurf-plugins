---
trigger: always_on
description: This file is for any AI coding agent working on this repository:
---

# AGENTS.md — Ori Codebase Guide for AI Coding Agents

This file is for any AI coding agent working on this repository:
Claude Code, Cursor, Codex, Gemini, Copilot, or any other tool.

For Claude Code specifically: `CLAUDE.md` contains the full architectural
specification including all data types, build order, and design decisions.
Read `CLAUDE.md` before writing any implementation code.

This file covers how to navigate and extend the codebase once you understand
the architecture.

---

## What this project is

Ori is an agentic IoT runtime. It reads physical sensor data and takes
autonomous actions based on LLM reasoning. It has an offline-capable safety
core and runs on a Raspberry Pi.

The key concept that every contributor must understand before touching code:

```text
Ori is NOT a monitoring system.
Ori is an agent that reasons about physical signals and acts on them.

Tier A actions  — always autonomous (alerts, logs)
Tier B actions  — autonomous by default (source switching, valve control)
Tier C actions  — approval required (breaker trips, equipment shutdown)
Tier D actions  — always autonomous, highest priority (safety cutoffs)
```

Every function, class, and test must be written with this framing in mind.
If you are writing code that makes Ori more passive, you are going in the
wrong direction.

---

## Repository layout

```text
ori/                    Python package — the runtime
├── hal/                Layer 1: hardware adapters
├── network/            Layer 2: event bus, deduplication, data types
├── reasoning/          Layer 4: LLM tiers + action dispatcher
├── skills/             Layer 5: skill loader and sandbox
├── actions/            Action executors called by the dispatcher
├── state/              SQLite state store
├── config.py           ori.yaml loader
└── runtime.py          Main event loop — ties everything together

skills/                 Bundled skills (YAML + Python hooks)
tests/                  pytest test suite
```

---

## The five extension points

When someone asks you to add something new to Ori, it almost always fits into
one of these five patterns. Identify the pattern first, then implement.

---

### 1. Adding a new HAL adapter

**When:** A new sensor protocol or hardware type needs support.
**Where:** `ori/hal/new_adapter.py`
**Pattern:**

```python
from ori.hal.base import BaseAdapter, AdapterConnectionError, AdapterTimeoutError
from ori.network.events import SensorReading
import time

class NewProtocolAdapter(BaseAdapter):

    def __init__(self):
        self._connected = False

    @property
    def is_connected(self) -> bool:
        return self._connected

    async def connect(self, config: dict) -> None:
        # config keys come from ori.yaml sensors[n] block
        # Raise AdapterConnectionError if connection fails
        try:
            # initialise hardware connection here
            self._connected = True
        except Exception as e:
            raise AdapterConnectionError(f"Failed to connect: {e}") from e

    async def read(self, sensor_id: str) -> SensorReading:
        # Must complete within config['read_timeout_ms'] or raise AdapterTimeoutError
        return SensorReading(
            sensor_id=sensor_id,
            sensor_type='temperature',   # match what ori.yaml declares
            value=0.0,
            unit='celsius',
            timestamp=int(time.time() * 1000),
            quality=1.0,
            metadata={'source': 'new_protocol'}
        )

    async def close(self) -> None:
        self._connected = False
```

**Rules:**

- Wrap all hardware imports in `try/except ImportError` so the adapter
  fails gracefully on platforms where the library is unavailable
- Never raise anything except `AdapterConnectionError` or `AdapterTimeoutError`
- Always set `metadata['source']` to the protocol name
- Add `@pytest.mark.skipif` to all tests that require real hardware:

  ```python
  skip_if_no_hardware = pytest.mark.skipif(
      not os.path.exists('/dev/i2c-1'),
      reason="Hardware not available"
  )
  ```

- Register the adapter in `ori/config.py` protocol map so ori.yaml can
  reference it by name

SHARED HARDWARE RESOURCES:
If your adapter accesses a hardware bus that can be shared across
multiple sensor instances (I2C, SPI), use the reference-counted
singleton pattern established in ori/hal/i2c_adapter.py.

Module-level dicts for hardware singletons are a permitted exception
to the no-global-state rule — hardware pins are physical singletons.
The singleton must:

- Use a threading.Lock for all cache operations
- Increment a ref count on connect()
- Decrement and conditionally evict on close()
- Never call deinit() during eviction if other refs may exist
- Stay isolated to the adapter module — never expose the cache
  to layers above the HAL

CIRCUIT BREAKER REQUIREMENT:
Every adapter that polls hardware must utilize the hardware circuit breaker gracefully:

```py
    from ori.hal.base import HardwareCircuitBreaker

    def __init__(self, adapter_name: str, config: dict):
        self._breaker = HardwareCircuitBreaker(adapter_name, config)

    async def read(self, sensor_id):
        async with self._breaker:
            return await self._do_read(sensor_id)
```

---

### 2. Adding a new skill


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ori-platform/ori-runtime](https://github.com/ori-platform/ori-runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
