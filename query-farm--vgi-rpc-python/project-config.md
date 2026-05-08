---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**vgi-rpc** is a transport-agnostic RPC framework built on Apache Arrow IPC serialization. RPC interfaces are defined as Python Protocol classes; the framework derives Arrow schemas from type annotations and provides typed client proxies with automatic serialization/deserialization.

## Commands

```bash
# Run all tests (includes mypy type checking and ruff linting via pytest plugins)
pytest

# Run a single test
pytest tests/test_rpc.py::test_name

# Lint and format
ruff check vgi_rpc/ tests/
ruff format vgi_rpc/ tests/

# Type checking
mypy vgi_rpc/

# Coverage (80% minimum, branch coverage enabled)
pytest --cov=vgi_rpc
```

Uses `uv` as the package manager. Install dev dependencies with `uv sync --all-extras`.

Tests should complete in 50 seconds or less ALWAYS!

Discourage the use of Any types, check mypy strict type coverage and always try to improve it.

Before pushing changes make sure, mypy, ruff and tests pass.

Pay attention to mypy strict type checking make sure strict typing is preserved.

Verify "ty" type checking too.

The full process before committing code is

1. Run `uv run ruff format .` on all files
2. Run `uv run ruff check .` and resolve all errors
3. Run `uv run mypy vgi_rpc/` and resolve all errors
4. Run `uv run ty check vgi_rpc/` and resolve all errors
5. Run `uv run pytest` for all tests

**Always reformat before pushing.** CI runs lint before tests — unformatted code fails the pipeline immediately and wastes time.

## Architecture

### Core modules (`vgi_rpc/`)

- **`rpc/`** — The RPC framework package. Defines the wire protocol, method types (UNARY, STREAM), and the core classes: `RpcServer`, `RpcConnection`, `RpcTransport`, `PipeTransport`, `ShmPipeTransport`, `SubprocessTransport`, `StreamSession`, `AnnotatedBatch`, `OutputCollector`. Also defines `AuthContext` (frozen dataclass for authentication state), `CallContext` (request-scoped context injected into methods via `ctx` parameter), `_TransportContext` (contextvar bridge for HTTP auth), `RpcError`, and `VersionError`. Introspects Protocol classes via `rpc_methods()` to extract `RpcMethodInfo` (schemas, method type). Client gets a typed proxy from `RpcConnection`; server dispatches via `RpcServer.serve()`. The convenience function `run_server()` parses `sys.argv` for `--http`, `--host`, and `--port` flags — without `--http` it serves over stdin/stdout pipes (for `SubprocessTransport`); with `--http` it lazily imports `vgi_rpc.http.serve_http` and serves over HTTP. The `_debug.py` submodule provides wire protocol debug logging infrastructure: 6 logger instances under the `vgi_rpc.wire.*` hierarchy (`request`, `response`, `batch`, `stream`, `transport`, `http`) and 4 formatting helpers (`fmt_schema`, `fmt_metadata`, `fmt_batch`, `fmt_kwargs`). All log points use `isEnabledFor` guards for zero overhead when disabled.

- **`utils.py`** — Arrow serialization layer. `ArrowSerializableDataclass` mixin auto-generates `ARROW_SCHEMA` from dataclass field annotations and provides `serialize()`/`deserialize_from_batch()`. Handles type inference from Python types to Arrow types (including generics, Enum, Optional, nested dataclasses). Also provides low-level IPC stream read/write helpers, `IpcValidation` enum, and `ValidatedReader`.

- **`log.py`** — Structured log messages (`Message` with `Level` enum). Messages are serialized out-of-band as zero-row batches with metadata keys `vgi_rpc.log_level`, `vgi_rpc.log_message`, `vgi_rpc.log_extra`. Server methods access logging via the `CallContext` (see below).

- **`logging_utils.py`** — `VgiJsonFormatter`, a `logging.Formatter` subclass that serializes log records as single-line JSON. Not auto-imported; must be imported explicitly from `vgi_rpc.logging_utils`.

- **`metadata.py`** — Shared helpers for `pa.KeyValueMetadata`. Centralises well-known metadata key constants (`vgi_rpc.method`, `vgi_rpc.stream_state#b64`, `vgi_rpc.log_level`, `vgi_rpc.log_message`, `vgi_rpc.log_extra`, `vgi_rpc.server_id`, `vgi_rpc.request_version`, `vgi_rpc.location`, `vgi_rpc.shm_offset`, etc.) and provides encoding, merging, and key-stripping utilities used by `rpc/`, `http/`, `log.py`, `external.py`, `shm.py`, and `introspect.py`.

- **`introspect.py`** — Introspection support. Provides the built-in `__describe__` RPC method, `MethodDescription`, `ServiceDescription`, `build_describe_batch`, `parse_describe_batch`, and `introspect()`. Enabled on `RpcServer` via `enable_describe=True`.

- **`shm.py`** — Shared memory transport support. Provides `ShmAllocator`, `ShmSegment`, and pointer batch helpers for zero-copy Arrow IPC batch transfer between co-located processes. Used by `ShmPipeTransport`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Query-farm/vgi-rpc-python](https://github.com/Query-farm/vgi-rpc-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
