---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this
repository.

## Project Overview

A modern, type-safe, async Python library for controlling LIFX smart devices over the local network.
Built with Python's built-in `asyncio` for async/await patterns and features auto-generated protocol
structures from a YAML specification. Published on PyPI as `lifx-async` (`pip install lifx-async`).

**Python Versions**: 3.10, 3.11, 3.12, 3.13, 3.14 (tested on all versions via CI)
**Runtime Dependencies**: Zero - completely dependency-free!
**Async Framework**: Python's built-in `asyncio` (no external async library required)
**Test Isolation**: lifx-emulator-core runs embedded in-process for fast, cross-platform testing

## Essential Commands

### Development Setup

```bash
# Sync all dependencies (including dev)
uv sync

# Install only the core library (zero dependencies)
uv sync --no-dev
```

### Adding a dependency

```bash
# Add a runtime dependency (use sparingly - library is currently dependency-free!)
uv add some-package

# Add a development dependency
uv add --dev pytest-cov
```

### Testing

```bash
# Run all tests
uv run --frozen pytest

# Run specific test file
uv run pytest tests/test_devices/test_light.py -v

# Run with coverage
uv run pytest --cov=lifx --cov-report=html

# Verbose output
uv run --frozen pytest -v

# Run with emulator integration tests (requires lifx-emulator on PATH)
# Tests marked with @pytest.mark.emulator will be skipped if emulator is not available
uv run pytest
```

### Code Quality

```bash
# Format code
uv run ruff format .

# Lint with auto-fix
uv run ruff check . --fix

# Type check (strict Pyright validation)
uv run pyright
```

### Protocol Update

```bash
# Source: https://github.com/LIFX/public-protocol/blob/main/protocol.yml
# Regenerate Python protocol code
uv run python -m lifx.protocol.generator
```

### Products Registry Update

```bash
# Source: https://github.com/LIFX/products/blob/master/products.json
# Regenerate Python product registry
uv run python -m lifx.products.generator
```

### Documentation

```bash
# Serve documentation locally with hot reload
uv run zensical serve

# Build static documentation
uv run zensical build
uv run llmstxt-standalone build

# Deploy to GitHub Pages via the Documentation workflow
gh workflow run docs.yml
```

## Architecture

### Layered Architecture (Bottom-Up)

1. **Protocol Layer** (`src/lifx/protocol/`)

   - Auto-generated from `protocol.yml` using `generator.py`
   - `protocol_types.py`: Enums and field structures (HSBK, TileStateDevice, etc.)
   - `packets.py`: Packet classes with PKT_TYPE constants
   - `header.py`: LIFX protocol header (36 bytes)
   - `serializer.py`: Binary serialization/deserialization
   - `models.py`: Protocol data models (`Serial` dataclass, HEV types)
   - `base.py`: Base classes for protocol structures
   - **Focus on lighting**: Button and Relay items are automatically filtered during generation (not
     relevant for light control)
   - **Never edit generated files manually** - download updated `protocol.yml` from LIFX official
     repo instead

2. **Network Layer** (`src/lifx/network/`)

   - `transport.py`: UDP transport using asyncio
   - `discovery.py`: Device discovery via broadcast with `DiscoveredDevice` dataclass
   - `connection.py`: Device connection with retry logic and lazy opening
   - `message.py`: Message building and parsing with `MessageBuilder`
   - `mdns/`: mDNS/DNS-SD discovery module (zero-dependency, stdlib only)
     - `discovery.py`: `discover_lifx_services()` and `discover_devices_mdns()`
     - `dns.py`: DNS wire format parser for PTR, SRV, A, TXT records
     - `transport.py`: `MdnsTransport` class for multicast UDP
     - `types.py`: `LifxServiceRecord` dataclass
   - Lazy connection opening (auto-opens on first request)

3. **Device Layer** (`src/lifx/devices/`)

   - `base.py`: Base `Device` class with common operations: `from_ip()`, label, power, version, info
   - `light.py`: `Light` class (color control, effects: pulse, breathe, waveforms)
   - `hev.py`: `HevLight` class (Light with HEV anti-bacterial cleaning cycle control)
   - `infrared.py`: `InfraredLight` class (Light with infrared LED control for night vision)
   - `multizone.py`: `MultiZoneLight` for strips/beams (zone-based color control)
   - `matrix.py`: `MatrixLight` for matrix devices (2D pixel control: tiles, candle, path)
   - `ceiling.py`: `CeilingLight` class (extends `MatrixLight` with independent uplight/downlight component control for LIFX Ceiling products)
   - State caching with configurable TTL to reduce network traffic

4. **High-Level API** (`src/lifx/api.py`)

   - `discover()`: Async generator yielding devices via UDP broadcast
   - `discover_mdns()`: Async generator yielding devices via mDNS (faster, single query)
   - `find_by_serial()`: Find specific device by serial number
   - `find_by_label()`: Async generator yielding devices matching label (exact or substring)
   - `find_by_ip()`: Find device by IP address using targeted broadcast
   - `DeviceGroup`: Batch operations (set_power, set_color, etc.)
   - `LocationGrouping` / `GroupGrouping`: Organizational structures for location/group-based grouping

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Djelibeybi/lifx-async](https://github.com/Djelibeybi/lifx-async) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
