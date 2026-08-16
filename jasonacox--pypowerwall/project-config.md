---
trigger: always_on
description: This document provides guidelines for AI code agents (Claude, Copilot, etc.) working on this codebase. Read [DESIGN.md](DESIGN.md) first for architecture; this file is the hands-on companion.
---

# AGENTS.md - AI Agent Guidelines for pypowerwall

This document provides guidelines for AI code agents (Claude, Copilot, etc.) working on this codebase. Read [DESIGN.md](DESIGN.md) first for architecture; this file is the hands-on companion.

## Quick Reference

```bash
# Run tests (unit + proxy; skips live-hardware tests)
pytest -m "not live"

# Run tests without coverage output (faster inner loop)
pytest -m "not live" --no-cov

# Lint (CI runs pylint across Python 3.9-3.13)
pylint pypowerwall proxy/server.py

# Run the proxy locally
python3 proxy/server.py

# Run against the simulator instead of real hardware
cd pwsimulator && docker build -t pwsimulator . && docker run -p 443:443 pwsimulator
# then: PW_HOST=localhost PW_PASSWORD=password PW_EMAIL=me@example.com python3 example.py

# CLI smoke test
python3 -m pypowerwall version
```

## The Prime Directive: Never Break the Public API

This library and its proxy are depended on by [Powerwall-Dashboard](https://github.com/jasonacox/Powerwall-Dashboard) and many other consumers running unattended 24/7. That means:

- **Method signatures**: existing positional/keyword parameters keep their names, order, and defaults. Add new parameters at the end with defaults. Legacy quirks like the `jsonformat` kwarg and the deprecated `type` argument to `grid_status()` stay.
- **Return shapes**: including *failure* shapes. If a method returns `None` on error today, it keeps returning `None` — not an exception, not `{}`.
- **Proxy HTTP responses**: status codes, JSON shapes, even oddities (some error paths return HTTP 200 with a JSON error body; Telegraf configs depend on current shapes). Fix bugs *behind* the interface, never by changing the interface.
- **The local `/api/...` URI namespace**: it is the internal lingua franca. Every non-local backend must answer the same URIs the local gateway does.

If a fix seems to require a breaking change, stop and flag it for the maintainer instead.

## Code Style

The library predates auto-formatters and is linted with **pylint** (not Black). Match the file you are editing:

- **Quotes**: single quotes dominate in `pypowerwall/`; double quotes dominate in `proxy/server.py`. Follow the surrounding file.
- **Line length**: ~100-120 characters is the norm. Don't reflow existing code.
- **Indentation**: 4 spaces.
- **Type hints**: public methods carry `Optional[Union[dict, list, str, bytes]]`-style annotations; internals are often untyped. Add hints to new public methods; don't retrofit whole files.
- **Naming**: `snake_case` functions; backend classes are `PyPowerwall<Mode>`; cache/state attributes use the `pw` prefix (`pwcache`, `pwcachetime`, `pwcacheexpire`, `pwcooldown`); proxy env vars are `PW_*`.
- **Comments**: this codebase favors generous explanatory comments, including root-cause narratives for workarounds (see the cloud 403 saga in `cloud/pypowerwall_cloud.py`). Keep that habit for anything non-obvious, especially firmware-version-specific behavior.
- **Module headers**: new modules get the banner-style docstring (description, "Author: Jason A. Cox", repo URL, Features/Functions lists) used everywhere else.

### Logging

Every module follows this pattern — do the same:

```python
import logging
log = logging.getLogger(__name__)

# ... and a module-level toggle:
def set_debug(toggle=True, color=True):
    ...
```

- Never log credentials, tokens, or cookies — not even at DEBUG level.
- Library code logs errors and returns `None`; it does not print. The CLI (`__main__.py`) and proxy do the printing.

### Error handling conventions (load-bearing — see DESIGN.md)

- Read methods: `log.error(...)` and **return `None`** on failure. Never raise to callers.
- `authenticate()`: **raise** on failure — this drives mode fallback in `connect()`.
- Invalid arguments: raise `ValueError` / `PyPowerwallInvalidConfigurationParameter`.
- Gateway rate limiting (429/503): set a cooldown, don't retry-loop.
- Guard every `lookup()`/dict access on gateway payloads — fields can be `None` or missing depending on firmware and hardware (PW2 vs PW3 vs cloud). Prefer `.get(key, default)` and `x or 0` over direct indexing.

## How to Add Things

### A new API endpoint (library)

Local mode passes any `/api/...` URI through automatically. The other three backends need explicit handlers — **all three, always**:

1. `pypowerwall/cloud/pypowerwall_cloud.py` — add `"/api/foo": self.get_api_foo` to `init_poll_api_map()`, implement `def get_api_foo(self, **kwargs)` using `self._site_api(...)`.
2. `pypowerwall/fleetapi/pypowerwall_fleetapi.py` — same pattern in its `init_poll_api_map()`.
3. `pypowerwall/tedapi/pypowerwall_tedapi.py` — same, pulling from `self.tedapi.get_status()/get_config()` with `lookup()`.
4. If a backend can't provide real data: add a constant to that backend's `mock_data.py`, decorate the handler with `@not_implemented_mock_data`, and return `json.loads(MOCK_CONSTANT)`. For partially-real data, deep-copy a template from `stubs.py` and fill it in.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jasonacox/pypowerwall](https://github.com/jasonacox/pypowerwall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
