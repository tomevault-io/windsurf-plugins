---
trigger: always_on
description: This document describes the architecture and development guidelines for future AI agents (Claude Code) and human developers.
---

# LaMetric Power Bridge - Developer Documentation

This document describes the architecture and development guidelines for future AI agents (Claude Code) and human developers.

---

## Architecture Overview

LaMetric Power Bridge is built with a **pluggable architecture** that clearly separates:

1. **Ingress (Sources)**: Data sources for power measurements
2. **Business Logic**: Data transformation and formatting
3. **Egress (Sinks)**: Output to LaMetric Time device

```
┌─────────────────────────────────────────────────────────────┐
│                    bridge.py (43 lines)                     │
│                     Orchestrator                            │
│                                                             │
│  1. Detect/select source (auto or --source CLI arg)        │
│  2. await source.connect()                                  │
│  3. async for reading in source.stream()                   │
│  4. await push_to_lametric(reading)                        │
└─────────────────────────────────────────────────────────────┘
           │                                  │
           ▼                                  ▼
┌──────────────────────┐          ┌──────────────────────┐
│   sources/           │          │   sinks/             │
│   (Ingress)          │          │   (Egress)           │
├──────────────────────┤          ├──────────────────────┤
│ base.py              │          │ lametric.py          │
│ - PowerReading       │─────────▶│ - push_to_lametric() │
│ - PowerSource        │          │ - HTTP POST          │
│   Protocol           │          │ - Icon selection     │
│                      │          │ - kW formatting      │
│ tibber.py            │          └──────────────────────┘
│ - TibberSource       │
│                      │          ┌──────────────────────┐
│ homewizard_v1.py     │          │   tests/             │
│ - HomeWizardV1Source │          ├──────────────────────┤
│   (v1 API - HTTP)    │          │ test_lametric.py     │
│                      │          │ test_tibber.py       │
│ homewizard_v2.py     │          │ test_homewizard_v1.py│
│ - HomeWizardV2Source │          │ conftest.py          │
│   (v2 API - WS)      │          └──────────────────────┘
│                      │
│ p1_serial.py         │
│ - P1SerialSource     │
│   (Serial)           │
└──────────────────────┘
```

---

## Core Concepts

### 1. PowerReading Dataclass

Uniform data contract between all sources and sinks:

```python
@dataclass
class PowerReading:
    power_watts: float      # Positive = consuming, negative = producing
    timestamp: str | None   # ISO8601, optional
```

### 2. PowerSource Protocol

Duck-typed interface (no ABC, no forced inheritance):

```python
class PowerSource(Protocol):
    async def connect(self) -> None:
        """Bootstrap: auth, discovery, setup"""
        ...

    async def stream(self) -> AsyncIterator[PowerReading]:
        """Yield PowerReading objects, handle auto-reconnect"""
        ...
```

**Why Protocol?**
- Explicit interface without boilerplate
- Duck typing continues to work
- Type checking with mypy/pyright
- Pythonic (no forced `class Foo(PowerSource)`)

### 3. Context Manager Pattern

All sources implement async context manager for automatic resource cleanup:

```python
async with HomeWizardV1Source(host="192.168.2.87") as source:
    async for reading in source.stream():
        await push_to_lametric(reading)
# Client automatically closed on exit
```

**Advantages**:
- Guaranteed cleanup (httpx client, websockets, etc)
- Pythonic async pattern
- No manual `aclose()` calls needed
- Also works with old `await source.connect()` pattern (backwards compatible)

**Implementation**:
```python
class YourSource:
    async def __aenter__(self):
        await self.connect()
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        # Cleanup resources (close clients, etc)
        if self.client:
            await self.client.aclose()
```

### 4. Single Responsibility

- **Sources**: Only fetch data and convert to PowerReading
- **Sinks**: Only format data and send
- **Bridge**: Only orchestration (no business logic)

---

## File Structure

```
lametric-power-bridge/
├── bridge.py                    # Orchestrator (43 lines)
├── sources/                     # Ingress modules
│   ├── __init__.py
│   ├── base.py                 # PowerReading + PowerSource Protocol
│   ├── tibber.py               # Tibber WebSocket implementation
│   ├── homewizard_v1.py        # HomeWizard P1 Meter (v1 HTTP API)
│   ├── homewizard_v2.py        # HomeWizard P1 Meter (v2 WebSocket)
│   └── p1_serial.py            # DSMR P1 Serial
├── sinks/
│   ├── __init__.py
│   └── lametric.py             # LaMetric formatting + HTTP push
├── tests/
│   ├── __init__.py
│   ├── conftest.py             # pytest fixtures
│   ├── test_lametric.py        # Sink tests (18 tests)
│   ├── test_tibber.py          # Tibber source tests (3 tests)
│   ├── test_homewizard_v1.py   # HomeWizard v1 tests (7 tests)
│   ├── test_homewizard_v2.py   # HomeWizard v2 tests (8 tests)
│   ├── test_p1_serial.py       # P1 Serial tests (11 tests)
│   └── test_bridge.py          # Bridge logic tests (5 tests)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spacebabies) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
