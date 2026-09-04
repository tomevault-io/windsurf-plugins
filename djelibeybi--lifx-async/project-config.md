---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this
repository.

## Project Overview

A modern, type-safe, async Python library for controlling LIFX smart devices over the local network.
Built with Python's built-in `asyncio` for async/await patterns and features auto-generated protocol
structures from a YAML specification. Published on PyPI as `lifx-async` (`pip install lifx-async`).

**Python Versions**: 3.10, 3.11, 3.12, 3.13, 3.14 (tested on all versions via CI)
**Runtime Dependencies**: Zero - completely dependency-free!
**Async Framework**: Python's built-in `asyncio` (no external async library required)
**Test Isolation**: lifx-emulator-core runs embedded in-process for fast, cross-platform testing

## Privacy and Hardware Identifiers

- Never commit personally identifiable information or live infrastructure identifiers. This
  includes real device serials or MAC addresses, IP addresses, local hostnames, account names and
  raw hardware-discovery output.
- Committed evidence must replace every live serial and address with a stable, format-preserving
  pseudonym from the operator's private mapping. Keep that mapping outside this repository; never
  copy it into source, planning artefacts, logs, prompts, reports or agent memory.
- When a check must establish that evidence came from an approved physical device, validate the raw
  result locally against the external mapping first. Commit only the pseudonymised artefact and
  retain the identity assertion as operator-controlled evidence outside the repository.
- Raw probe output may exist transiently in the local terminal or an ignored temporary file, but it
  must be sanitised before being copied into any tracked file. Tools such as
  `scripts/ipv6_thread_probe.py` may emit live identifiers, so treat their output as private by
  default.
- Tests and documentation must use clearly synthetic identifiers and non-live example addresses.
  Preserve stable aliases across related artefacts so evidence remains correlatable without
  revealing the underlying device or network.
- Before staging hardware or network evidence, inspect the staged diff for serials, MAC addresses,
  IP addresses, hostnames and other PII. If a raw identifier reaches a local commit, amend or rewrite
  it before any push; a later redaction commit is insufficient because the identifier remains in
  history.

## Git Commits

- Use Conventional Commit messages.
- Do not use GSD phase or plan metadata as a Conventional Commit scope. Scopes such as `10`,
  `phase-10` and `10-07` describe planning bookkeeping rather than a repository component. Omit the
  scope when no stable repository-specific scope applies.
- Always use `git commit -S -s` so every commit is GPG-signed and carries the developer sign-off.

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

### Running the measurement scripts

`scripts/measurement_support.py` owns the shared discovery, request-observation, capture
and restore primitives. The three scripts that import it must be run as modules, not as
files:

```bash
uv run --frozen python -m scripts.thread_revalidation <subcommand>
uv run --frozen python -m scripts.ipv6_thread_probe
uv run --frozen python -m scripts.measure_merged_discovery
```

`from scripts.measurement_support import ...` resolves only when the repository root is on
`sys.path`. `python -m` puts it there; running the file directly puts `scripts/` there
instead and fails with `ModuleNotFoundError: No module named 'scripts'`. Adding

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Djelibeybi/lifx-async](https://github.com/Djelibeybi/lifx-async) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
