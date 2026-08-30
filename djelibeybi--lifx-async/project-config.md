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

# Run with emulator integration tests (lifx-emulator-core is a required dev dependency)
# Use --disable-emulator to skip the normal embedded-emulator suite explicitly
uv run pytest
```

Pytest retries each test once, with no delay, only when it raises the exact
`LifxTimeoutError`, `LifxConnectionError`, or `LifxNetworkError` type. Assertion
failures and all other exceptions fail immediately. The suite-wide 60-second
timeout covers two complete default 16-second request attempts; emulator tests
receive a 120-second timeout. The targeted IPv6 emulator lookup retains two
retries with a one-second delay on Windows because its socket and scheduler
window can outlast one immediate retry. That targeted override also admits the
assertion-shaped no-response result. It is applied during collection only on
Windows; elsewhere the test keeps the global one immediate network retry and
ordinary assertion failures still fail immediately.

### Code Quality

```bash
# Format code
uv run ruff format .

# Lint with auto-fix
uv run ruff check . --fix

# Type check (Pyright, standard mode)
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

### Theme Data Update

```bash
# Source: data/themes.jsonl (committed; no network, no device required)
# Regenerate the theme data module
uv run scripts/generate_theme_data.py
```

The generator lives in `scripts/`, not the package: its input sits outside `src/` and is
deliberately not shipped in the wheel. CI regenerates and diffs `src/lifx/theme/data.py`
on every change to `data/**`.

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
   - `discovery.py`: Device discovery via IPv4 broadcast or targeted IPv4/IPv6 datagrams with `DiscoveredDevice` dataclass
   - `connection.py`: Device connection with retry logic and lazy opening
   - `message.py`: Message building and parsing with `MessageBuilder`
   - `mdns/`: mDNS/DNS-SD discovery module (zero-dependency, stdlib only)
     - `discovery.py`: Per-call service-record assembly and supported device construction.
       Each sweep sends an initial DNS-SD PTR service query, may retransmit that PTR query
       once at one second and once at three seconds within the caller's deadline, and
       assembles valid legacy-unicast replies during the quiet window. When a valid SRV
       target still lacks a usable address, it conditionally sends bounded A/AAAA follow-ups:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Djelibeybi/lifx-async](https://github.com/Djelibeybi/lifx-async) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
