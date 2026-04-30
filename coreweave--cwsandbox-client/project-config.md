---
trigger: always_on
description: This file provides guidance to AI coding assistants when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding assistants when working with code in this repository.

## Project Overview

Python client library for CoreWeave Sandbox - a remote code execution platform. The SDK provides a sync/async hybrid API for creating, managing, and executing code in containerized sandbox environments.

## Public API and Documentation

When adding, removing, or renaming public exports in `src/cwsandbox/__init__.py`, the API reference generator in the `coreweave/docs` repo needs its `MANIFEST_GROUPS` updated (in `scripts/cwsandbox-api-ref/generate.py`). Docstrings use Google style with `Examples:` and `Attributes:` sections for structured parsing by Griffe.

## Development Setup

See [DEVELOPMENT.md](DEVELOPMENT.md) for setup, workflow, and all development tasks.

## Architecture

### Core Classes

**`Sandbox`** (`_sandbox.py`): Main entry point with sync/async hybrid API. All methods return immediately; operations execute in background via `_LoopManager`.

Construction patterns:
```python
# Factory method (recommended)
sb = Sandbox.run("echo", "hello")  # Returns immediately
result = sb.exec(["echo", "more"]).result()  # Block for result
sb.stop().result()  # Block for completion

# Context manager (recommended for most use cases)
with Sandbox.run() as sb:  # Default command keeps sandbox alive
    result = sb.exec(["echo", "hello"]).result()
# Automatically stopped on exit

# Streaming output before getting result
with Sandbox.run() as sb:
    process = sb.exec(["echo", "hello"])
    for line in process.stdout:  # Stream lines as they arrive
        print(line, end="")
    result = process.result()  # Get final ProcessResult

# Async context manager
async with Sandbox.run() as sb:
    result = await sb.exec(["echo", "hello"])
```

Key methods:
- `run(*args, **kwargs)`: Create and start sandbox, return immediately. Accepts advanced configuration kwargs (see below).
- `start()`: Send start request, return `OperationRef[None]`. Call `.result()` to block until backend accepts.
- `wait()`: Block until RUNNING status, returns self for chaining
- `wait_until_complete(timeout=None, raise_on_termination=True)`: Wait until terminal state (COMPLETED, FAILED, TERMINATED), return `OperationRef[Sandbox]`. Polls through TERMINATING automatically. Call `.result()` to block or `await` in async contexts. Set `raise_on_termination=False` to handle externally-terminated sandboxes without raising `SandboxTerminatedError`.
- `exec(command, cwd=None, check=False, timeout_seconds=None, stdin=False)`: Execute command, return `Process`. Call `.result()` to block for `ProcessResult`. Iterate `process.stdout` before `.result()` for real-time streaming. Set `check=True` to raise `SandboxExecutionError` on non-zero returncode. Set `cwd` to an absolute path to run the command in a specific working directory (implemented via shell wrapping, requires /bin/sh in container). Set `stdin=True` to enable stdin streaming via `process.stdin`.
- `shell(command=None, *, width=None, height=None)`: Start an interactive TTY session, return `TerminalSession`. Always allocates a TTY and enables stdin. Output is raw bytes (merged stdout/stderr) with no buffering — safe for long-running interactive sessions. Defaults to `["/bin/bash"]`.
- `stream_logs(*, follow=False, tail_lines=None, since_time=None, timestamps=False)`: Stream logs from the sandbox's main process (PID 1), return `StreamReader[str]`. Only captures stdout/stderr from the command passed to `Sandbox.run()` — output from `exec()` commands is **not** included. Set `follow=True` for continuous streaming (like `tail -f`). Uses bounded queues for backpressure in follow mode.
- `read_file(path)`: Return `OperationRef[bytes]`
- `write_file(path, content)`: Return `OperationRef[None]`
- `stop(snapshot_on_stop=False, graceful_shutdown_seconds=10.0, missing_ok=False)`: Stop sandbox and return `OperationRef[None]`. The sandbox transitions through TERMINATING (grace period) before reaching a terminal state (COMPLETED or FAILED). The returned OperationRef resolves when the backend confirms a terminal state, not just when the stop RPC succeeds. Multiple callers share the same stop task. Raises `SandboxError` on failure. Set `snapshot_on_stop=True` to capture sandbox state before shutdown. Set `missing_ok=True` to suppress `SandboxNotFoundError`.
- `get_status()`: Fetch fresh status from API (sync). Returns cached status for terminal sandboxes (COMPLETED, FAILED, TERMINATED) since terminal states are immutable. TERMINATING is non-terminal and always fetches fresh status.

Properties:
- `status`: Cached status from last API call (use `get_status()` for fresh)
- `status_updated_at`: When status was last fetched
- `sandbox_id`, `runner_id`, `profile_id`, `runner_group_id`, `returncode`, `started_at`
- `resource_requests`, `resource_limits` - Confirmed resources from start response (None for discovered sandboxes)

Advanced configuration kwargs (for `run()`, `Session.sandbox()`, and `@session.function()`):
- `resources` - Resource configuration via `ResourceOptions`, nested dict, or legacy flat dict (CPU, memory, GPU)
- `mounted_files` - Files to mount into the sandbox
- `s3_mount` - S3 bucket mount configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coreweave/cwsandbox-client](https://github.com/coreweave/cwsandbox-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
