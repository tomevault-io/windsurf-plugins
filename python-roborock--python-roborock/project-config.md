---
trigger: always_on
description: `python-roborock` is an asynchronous (`asyncio`) device integration library supporting multi-protocol Roborock vacuum and home appliances (V1 JSON-RPC over AES, B01 Tuya DPS for Q7/Q10, and A01 Tuya DPS for Dyad/Zeo). While Home Assistant Core is a primary downstream consumer, `python-roborock` is an independent client library whose public abstractions must remain general-purpose and consumer-agnostic.
---

# Repository Engineering Guidelines: python-roborock

## Overview & Scope
`python-roborock` is an asynchronous (`asyncio`) device integration library supporting multi-protocol Roborock vacuum and home appliances (V1 JSON-RPC over AES, B01 Tuya DPS for Q7/Q10, and A01 Tuya DPS for Dyad/Zeo). While Home Assistant Core is a primary downstream consumer, `python-roborock` is an independent client library whose public abstractions must remain general-purpose and consumer-agnostic.

This document serves as the authoritative, single source of truth for engineering conventions, architectural standards, and automated code reviews across all contributors and coding agents.

---

## Environment & Tooling Standards
- **Target**: Python 3.11+, Hatchling build backend, `uv` package manager.
- **Linters & Formatters**: Ruff (line length 120), Mypy (`check_untyped_defs = true`), Codespell.
- **Testing**: pytest (`pytest-asyncio`).

### Key Developer Commands
```bash
# Environment setup
uv venv && uv sync

# Run tests
uv run pytest
uv run pytest tests/devices/traits/v1/test_status.py
uv run pytest tests/protocols/test_v1_protocol.py

# Lint and typecheck
uv run pre-commit run --all-files
# or directly:
uv run ruff check roborock tests
uv run mypy roborock tests
```

---

## Core Architectural Hierarchy & Review Priorities
Evaluate every change against the repository's three core layers in strict priority order:

1. **Priority 1: Public Trait APIs & Consumer Contracts** (Highest Priority)
   - Clean, stable user-facing abstractions decoupled from wire protocol details, Tuya DPS keys, encryption keys, or transport sockets.
   - Strongly typed with concrete models subclassing `RoborockBase`; never expose raw protocol dictionaries or `Any` on public boundaries.

2. **Priority 2: Data Lifecycle & State Management**
   - Simple, linear data flow. Keep side effects visible: helper functions MUST be pure transformations that compute and return values rather than mutating object state as a side effect.
   - Exhaustive lifecycle teardown: all listeners, background tasks, and channels cleanly unhooked in `RoborockDevice.close()`.
   - Concurrency: 1:1 request/response matching across async push channels uses `asyncio.Future` correlated by protocol request/sequence ID (`request_id` for V1 RPC, `msg_id` for B01/Tuya).
   - Transform data in render pipelines; never mutate cached telemetry state or raw packets for presentation effects.

3. **Priority 3: Wire Protocol Parsers, Codecs & Cryptography**
   - Encapsulate cryptography (AES, MD5), protocol framing, and raw sockets to `roborock.protocols` and `roborock.devices.transport` / `roborock.devices.rpc`.
   - Keep protocol families (V1 JSON-RPC, B01 Tuya DPS, A01 Tuya DPS) strictly isolated; never conflate schemas across families.
   - Enforce enum fallback resilience (`RoborockEnum` with lowercase `unknown = -1` or `RoborockModeEnum.from_code_optional()`); never crash on unexpected firmware codes.

---

## Detailed Engineering Conventions

### 1. Typing & Data Models
- **Subclass `RoborockBase`**: Define structured domain and wire data models as `@dataclass` subclassing `RoborockBase` (`from_dict`, `as_dict`). Avoid `TypedDict` or loose dicts. (Binary protocol packets, transport message envelopes, and map layers are exempt). The existing frozen `Q10RoborockPoint` coordinate value is also exempt: it must retain immutability and hashability, and Python disallows frozen dataclass inheritance from the non-frozen `RoborockBase`. This exception does not extend to other domain models.
- **Enum Fallback Resilience**: All enums representing device status, firmware modes, error codes, and wire protocol integer codes MUST inherit from `RoborockEnum` (defining a lowercase `unknown = -1` or `0` member) or `RoborockModeEnum` (using `from_code_optional()`). Internal enums not decoding unknown firmware codes remain standard `Enum`/`StrEnum`.
- **Strongly Type What You Know; Contain `Any` to the Wire**: Public trait APIs, method signatures, properties, and domain models MUST declare concrete types. `Any` is accepted only where the underlying wire protocol is dynamic or polymorphic (Tuya DPS maps, low-level RPC dispatch, serialization helpers, evolving cloud schemas).
- **Avoid Forward References & `TYPE_CHECKING`**: Avoid stringified forward references (`"ClassName"`) and `if typing.TYPE_CHECKING:` guards wherever possible. They typically indicate circular dependencies or coupling that should be refactored by extracting shared models.
- **Explicit Parameters**: Do not mark arguments or dataclass fields as `Optional[...]` or default them to `None` if they are always required by the protocol or caller.
- **Trust Type Annotations**: Prohibit defensive runtime `isinstance` checks on statically typed parameters in business and trait logic. (Dynamic wire payload unpacking and type narrowing in `from_dict` or RPC deserializers is legitimate and expected).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Python-roborock/python-roborock](https://github.com/Python-roborock/python-roborock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
