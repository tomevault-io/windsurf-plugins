---
trigger: always_on
description: > **End-user cheatsheet for AI-assisted consumption:** [`llms.txt`](llms.txt) — use that one if you're writing code *against* this SDK.
---

# AGENTS.md — UNICORN Binance REST API

> **End-user cheatsheet for AI-assisted consumption:** [`llms.txt`](llms.txt) — use that one if you're writing code *against* this SDK.
> **This file** is for AI agents working *on* this repo itself.

## Why things are the way they are

See [`context/index.md`](context/index.md) before making non-trivial changes — it points to the reasoning behind design decisions, rejected alternatives, and constraints that aren't visible in the code. If `AGENTS.local.md` exists in this repo, that's personal/local notes, not relevant to anyone else.

## Planning & Backlog

Open development tasks and decisions are tracked in **[TASKS.md](TASKS.md)**.

---

## Project Overview

Python SDK (MIT License) for the Binance REST APIs. Covers Spot, Margin, Isolated Margin, Futures (USDT-M & Coin-M), Testnet variants, Binance.US and TRBinance.com. Forked from python-binance (Sam McHardy), heavily extended and maintained by Oliver Zehentleitner.

**Current Version:** 2.12.0.dev  
**Next Release:** 2.12.0  
**Python Compatibility:** 3.9 – 3.14  
**Author:** Oliver Zehentleitner  
**PyPI:** `unicorn-binance-rest-api`  
**Part of:** [UNICORN Binance Suite](https://github.com/oliver-zehentleitner/unicorn-binance-suite)

---

## Directory Structure

```
unicorn_binance_rest_api/
    manager.py          # Core class BinanceRestApiManager — all REST endpoints
    enums.py            # Exchange constants (order types, intervals, sides, etc.)
    exceptions.py       # Custom exceptions
    helpers.py          # Utility functions (date_to_milliseconds, interval_to_milliseconds)
    __init__.py         # Exports BinanceRestApiManager + enums + exceptions + helpers

unittest_binance_rest_api.py   # Unit tests (run in CI)
dev/                           # Local dev/integration tests — NOT run in CI
examples/                      # Usage examples
docs/                          # Pre-built HTML documentation (Sphinx)
dev/sphinx/                    # Sphinx source for rebuilding docs
```

---

## Supported Exchanges

| Exchange String | Description |
|---|---|
| `binance.com` | Binance.com Spot (default) |
| `binance.com-testnet` | Binance.com Spot Testnet |
| `binance.com-margin` | Binance.com Cross Margin |
| `binance.com-margin-testnet` | Binance.com Cross Margin Testnet |
| `binance.com-isolated_margin` | Binance.com Isolated Margin |
| `binance.com-isolated_margin-testnet` | Binance.com Isolated Margin Testnet |
| `binance.com-futures` | Binance.com USDT-M Futures |
| `binance.com-futures-testnet` | Binance.com USDT-M Futures Testnet |
| `binance.com-coin_futures` | Binance.com Coin-M Futures |
| `binance.com-portfolio_margin` | Binance.com Portfolio Margin (listenKey management only, see below) |
| `binance.us` | Binance.US |
| `trbinance.com` | TRBinance.com |

Portfolio Margin (`binance.com-portfolio_margin`) is scoped to the PAPI
`listenKey` endpoints only — `portfolio_margin_stream_get_listen_key()`,
`portfolio_margin_stream_keepalive()`, `portfolio_margin_stream_close()`
(`PAPI_URL = https://papi.binance.com/papi`). Why the scope is this narrow:
[`context/portfolio-margin.md`](context/portfolio-margin.md).

---

## Dependencies

Managed in `requirements.txt`, `setup.py`, `pyproject.toml`, `environment.yml`, and `meta.yaml` — **all five must be kept in sync manually** (why, and a real drift incident: [`context/history.md`](context/history.md)):

- `requests>=2.32.4` — HTTP client
- `certifi>=2025.6.15` — TLS certificates
- `cryptography>=45.0.4` — signature/encryption
- `pyOpenSSL` — SSL support
- `service-identity` — SSL identity verification
- `colorama` — colored terminal output (see [`context/testing.md`](context/testing.md) for the `wrap=False` gotcha)
- `dateparser` — date string parsing
- `regex` — regex utilities
- `PySocks` — SOCKS5 proxy support
- `Cython` — C extension compilation (release builds only)

---

## Workflow

Every change must be accompanied by updates to **all affected locations**:
- `README.md` — Python version range, feature descriptions, endpoint lists
- `docs/` — pre-built Sphinx HTML; rebuild via `dev/sphinx/create_docs.sh` if content changed
- `CHANGELOG.md` — entry for every user-visible change
- Dependency files — `requirements.txt`, `setup.py`, `pyproject.toml`, `environment.yml`, `meta.yaml` (all five in sync)

Never open a PR without checking README.md and docs for stale information.

---

## Running Tests

```bash
# Unit tests with coverage (this is what CI runs)
coverage run --source unicorn_binance_rest_api unittest_binance_rest_api.py

# Unit tests without coverage
python unittest_binance_rest_api.py
```

Tests in `dev/` are local integration tests that require live Binance credentials — they are **not run in CI**. CI initializes against `binance.us`, not `binance.com` — see [`context/testing.md`](context/testing.md).

---

## Build & Packaging

Development and testing use **plain Python** — no Cython compilation needed during development.

Cython compilation only happens for **release builds**:

```bash
python setup.py bdist_wheel
```

`setup.py` also auto-generates `.pyi` stub files via `stubgen` and moves them into the package directory.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oliver-zehentleitner/unicorn-binance-rest-api](https://github.com/oliver-zehentleitner/unicorn-binance-rest-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
