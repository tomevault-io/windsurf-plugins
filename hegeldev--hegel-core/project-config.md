---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hegel is a universal property-based testing protocol. A Python server (powered by Hypothesis) communicates with language-specific client libraries over the server's stdin/stdout. This repository contains the Python core server.

## Build & Test Commands

```bash
uv sync --group dev             # Install with dev dependencies
just ci                         # Run all CI checks (lint + typecheck + coverage)
just test                       # Run tests
just coverage                   # Run tests with coverage (must be 100%)
just format                     # Auto-fix lint + format (ruff + shed)
just check                      # typecheck + format + coverage
uv run pytest tests             # Run all tests directly
uv run pytest tests/test_schema.py  # Run single test file
uv run pytest -k test_name     # Run tests matching pattern
uv run mypy src/                # Type check (targets Python 3.14)
```

100% branch test coverage is required (`fail_under = 100`). Uses `uv` as package manager, `ruff` + `shed` for formatting. Coverage omits `src/hegel/conformance.py`.

Coverage runs twice in CI: once normally and once with `ANTITHESIS_OUTPUT_DIR` set (which switches the Hypothesis backend to `hypothesis-urandom`), then combines results. Both runs must be covered.

## Architecture

### Client-Server Communication

1. **Client** spawns the `hegel` CLI as a subprocess and communicates over its stdin/stdout
2. A single persistent connection supports multiple test executions
3. **Hypothesis ConjectureRunner** drives test execution on the server side, including shrinking

### Protocol

Binary protocol over stdin/stdout with CBOR-encoded payloads:
- 20-byte header + variable payload + terminator byte: magic `0x4845474C` (HEGL), CRC32, stream ID, message ID, payload length
- Stream 0 is the control stream; odd-numbered streams are client-created, even-numbered are server-created
- Reply bit (`1 << 31`) in message ID distinguishes requests from replies
- Commands: `generate`, `start_span`, `stop_span`, `target`, `mark_complete`, `new_collection`, `collection_more`, `collection_reject`, `new_pool`, `pool_add`, `pool_generate`, `pool_consume`, `new_state_machine`, `state_machine_next_rule`
- `PROTOCOL_VERSION` (in `protocol/connection.py`) is reported to clients during the handshake. **Bump it for any change to the wire protocol** — adding a command or a field, renaming or removing one, or otherwise breaking compatibility.

### Module Overview

- `__main__.py` - CLI entry point (`hegel` command via click). Speaks the protocol over stdin/stdout.
- `server.py` - Drives test execution via Hypothesis `ConjectureRunner` with a `ThreadPoolExecutor`. Contains `HegelState` (per-test-run state) and `_run_test` (orchestrates a full test including shrinking and final replay)
- `protocol/` - Binary protocol package:
  - `packet.py` - Wire format: `Packet` dataclass, `read_packet`/`write_packet` for serialization with CRC32 checksums
  - `connection.py` - `Connection` class: manages socket, reader thread, stream registry, handshake. Thread-safe writes via lock
  - `stream.py` - `Stream` class: per-stream packet queues (requests/replies), `send_request`/`handle_requests` for CBOR request-reply pattern, `PendingRequest` future
  - `utils.py` - `StreamId`/`MessageId` newtypes, `ProtocolError`/`RequestError` exceptions, `STREAM_TIMEOUT`
- `schema.py` - JSON Schema to Hypothesis strategy conversion (cached by SHA1 hash in `FROM_SCHEMA_CACHE`)
- `test_server.py` - Simplified server for error simulation testing (activated via `HEGEL_PROTOCOL_TEST_MODE`). Used by library conformance tests to validate error handling
- `conformance.py` - Framework for testing library implementations against specification (`ConformanceTest` base class with `__init_subclass__` auto-registration)
- `utils.py` - `UniqueIdentifier` sentinel factory, `not_set` sentinel

### Server Execution Flow

1. Client sends `run_test` on the control stream (stream 0)
2. `server.py` creates a `ConjectureRunner` and a per-test stream
3. For each test case, a `test_case_stream` is created and the client is notified
4. Client sends commands (`generate`, `start_span`/`stop_span`, `target`, `mark_complete`) on the test case stream
5. `generate` calls `data.draw(from_schema(schema))` to produce values
6. After all test cases, interesting (failing) examples are replayed with `is_final=True`

### Generator Modes

1. **Schema Composition** (preferred): Compose JSON schemas, single protocol request. Generators that have schemas are called "basic generators."
2. **Compositional Fallback**: Multiple requests wrapped in spans when schemas unavailable (after `map`/`filter` on non-basic generators, or `flatmap`)

Key insight: `map()` on a basic generator preserves the schema by composing the transform function, rather than losing it. This is the central optimization across all libraries.

### Key Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hegeldev/hegel-core](https://github.com/hegeldev/hegel-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
