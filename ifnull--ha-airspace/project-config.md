---
trigger: always_on
description: Project-specific guidance for Claude Code working on `ha-airspace`.
---

# CLAUDE.md

Project-specific guidance for Claude Code working on `ha-airspace`.

> Always read `DESIGN.md` first — it's the source of truth for architecture, scope, and phase boundaries. This file covers conventions and workflow.

---

## Project at a glance

`ha-airspace` is a Python service that consumes `aircraft.json` from one or more ADS-B receivers (dump1090, readsb, dump978-fa), enriches with reference databases, applies tagging/alert rules, and publishes to MQTT for Home Assistant consumption.

Distribution targets: HA add-on (primary), Docker image, pip package — same codebase.

**Non-goals to remember:** not a feeder, not a replacement for dump1090, not a website, not a HACS integration. If a change pushes toward any of those, stop and check in.

---

## Tech stack

- **Python 3.12+**. Use modern syntax (`match`, `|` unions, `Self`, parameterized generics without `typing.`).
- **Async-first.** `asyncio` throughout. Receivers, MQTT, DB refresh — all async. No threading except for CPU-bound DB parsing in an executor.
- **`httpx`** for HTTP (async, supports HTTP/2, sane timeouts).
- **`aiomqtt`** for MQTT (asyncio-native paho wrapper).
- **`pydantic` v2** for config schema validation. Strict mode; reject unknown fields with helpful errors.
- **`structlog`** for logging. JSON output in production, console renderer in dev.
- **`uv`** for dependency management and virtualenvs. `pyproject.toml` is canonical; no `requirements.txt`.
- **`ruff`** for lint + format (replaces black, isort, flake8).
- **`pytest`** + **`pytest-asyncio`** for tests.
- **`mypy --strict`** for type checking.

Don't introduce new dependencies without flagging it. The runtime footprint matters — this ships as a HA add-on and people run it on Raspberry Pis.

---

## Code conventions

### Style

- Format and lint with `ruff`. The repo's `pyproject.toml` is the source of truth — don't override locally.
- Line length: 100. Not 80, not 120.
- Type hints on every function signature, including tests. `Any` is a code smell; if you reach for it, leave a comment explaining why.
- `from __future__ import annotations` at the top of every module. Forward references everywhere, no exceptions.
- Imports: stdlib, third-party, first-party — three groups separated by blank lines. `ruff` enforces this.
- Prefer `dataclasses` (frozen where possible) over plain classes for data containers. Use `pydantic.BaseModel` only at config boundaries where validation matters.

### Naming

- Modules: `snake_case`, short, no plurals (`receiver.py` not `receivers.py`).
- Classes: `PascalCase`. Abstract base classes don't need an `Abstract` prefix; the abstract methods make it obvious.
- Functions/methods: `snake_case`, verb-first (`fetch_aircraft`, not `aircraft_fetch`).
- Constants: `SCREAMING_SNAKE_CASE` at module level. No `Constants` class.
- Private: single leading underscore. Don't use double-underscore mangling.

### Async patterns

- Every entry point is `async def`. The only `def` functions are pure utilities.
- Use `asyncio.TaskGroup` (3.11+) for structured concurrency, not `asyncio.gather` with bare exception handling.
- Cancellation must be respected — never swallow `asyncio.CancelledError`.
- HTTP clients are long-lived (`httpx.AsyncClient` per receiver, reused across polls). Don't open a new client per request.
- Backoff with `tenacity` or hand-rolled exponential — but always cap retries and surface failures via the receiver's `health()` rather than crashing the loop.

### Error handling

- **Receivers fail in isolation.** A flaky receiver should never take down the merger. Catch broadly inside the receiver's poll loop, log, mark unhealthy, yield empty observations, keep going.
- **MQTT disconnects are recoverable.** Reconnect with backoff. Don't drop aircraft state on disconnect; republish on reconnect.
- **Config errors fail fast.** Bad config → exit non-zero with a clear message at startup. Don't try to "do your best."
- **Use exceptions for unexpected, return values for expected.** A receiver returning empty list is expected (transient outage). A receiver returning malformed JSON repeatedly is unexpected (raise → caught at receiver boundary → marked unhealthy).

### Logging

```python
import structlog
log = structlog.get_logger()

log.info("aircraft_observed", hex=obs.hex, receiver=obs.seen_by, distance_nm=dist)
log.warning("receiver_unhealthy", receiver=name, error=str(e), retry_in_s=backoff)
```

- Event names are `snake_case` past-tense or descriptive nouns. Not English sentences.
- Structured fields, not f-string interpolation. Makes log aggregation actually useful.
- Use `log.exception(...)` only when re-raising or terminating; otherwise log the error message in a field.

---

## Repo layout

```
ha-airspace/
├── DESIGN.md                  # architecture spec (READ FIRST)
├── CLAUDE.md                  # this file
├── README.md                  # user-facing
├── pyproject.toml
├── uv.lock
├── src/
│   └── ha_airspace/
│       ├── __init__.py
│       ├── __main__.py        # entry point
│       ├── config.py          # pydantic schema + loading
│       ├── models.py          # AircraftObservation, AircraftState, etc.
│       ├── receivers/
│       │   ├── __init__.py
│       │   ├── base.py        # ReceiverSource ABC

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ifnull/ha-airspace](https://github.com/ifnull/ha-airspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
