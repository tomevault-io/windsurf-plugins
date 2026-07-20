---
trigger: always_on
description: `pynetbox` is a Python API client library for NetBox. It provides a Pythonic interface to interact with NetBox's REST API and supports NetBox 3.3+ (pynetbox 6.7+). It is a standalone library — not a Django app or NetBox plugin — published to PyPI and used by automation tooling, scripts, and other Python projects to read and write NetBox data. Current version: `7.7.0` (see `pynetbox/__init__.py`).
---

# AGENTS.md — pynetbox

## Repository Overview

`pynetbox` is a Python API client library for NetBox. It provides a Pythonic interface to interact with NetBox's REST API and supports NetBox 3.3+ (pynetbox 6.7+). It is a standalone library — not a Django app or NetBox plugin — published to PyPI and used by automation tooling, scripts, and other Python projects to read and write NetBox data. Current version: `7.7.0` (see `pynetbox/__init__.py`).

## Tech Stack

- Python 3.10+ (tested on 3.12, 3.13, 3.14 in CI)
- `requests` + `urllib3` for HTTP (pins in `requirements.txt`)
- `packaging` for version comparisons
- `pytest` + `pytest-docker` for the test suite (unit and integration)
- `ruff` for linting (no `ruff.toml`; configured inline or with defaults)
- `mkdocs-material` + `mkdocstrings` for user-facing docs

Defer all version pins to `requirements.txt` and `requirements-dev.txt`. Package metadata lives in `setup.py` (uses `setuptools_scm` for version from git tags).

## Repository Map

```text
.
├── pynetbox/
│   ├── __init__.py              — Public API: Api, exceptions, __version__.
│   ├── core/
│   │   ├── api.py               — Api class: entry point, session management, app init.
│   │   ├── app.py               — App and PluginsApp: dynamic endpoint access.
│   │   ├── endpoint.py          — Endpoint: CRUD operations, filter validation.
│   │   ├── query.py             — Request: HTTP layer, threading, exceptions.
│   │   ├── response.py          — Record, RecordSet: deserialization, save/delete.
│   │   └── util.py              — Shared utilities.
│   └── models/
│       ├── circuits.py          — Circuit-specific Record subclasses.
│       ├── core.py              — DataSources, Jobs, ObjectChanges models.
│       ├── dcim.py              — DCIM models: Devices, Interfaces, Cables, tracing.
│       ├── extras.py            — Custom fields, tags, webhooks.
│       ├── ipam.py              — IP address management: IpAddresses, Prefixes, VLANs.
│       ├── mapper.py            — CONTENT_TYPE_MAPPER: content-type string → Record class.
│       ├── users.py             — User and permission models.
│       ├── virtualization.py    — Virtual machine models.
│       └── wireless.py          — Wireless endpoint models.
├── tests/
│   ├── conftest.py              — Pytest options: --netbox-versions, --no-cleanup, --url-override.
│   ├── fixtures/                — JSON response fixtures for unit tests (organised by app).
│   ├── test_api.py              — Api-level unit tests.
│   ├── test_app.py              — App/PluginsApp unit tests.
│   ├── test_circuits.py         — Circuits unit tests (older style).
│   ├── test_dcim.py             — DCIM unit tests (older style).
│   ├── test_tenancy.py          — Tenancy unit tests.
│   ├── test_users.py            — Users unit tests.
│   ├── test_virtualization.py   — Virtualization unit tests.
│   ├── test_wireless.py         — Wireless unit tests.
│   ├── util.py                  — Shared test helpers.
│   ├── unit/                    — Newer unit tests (mock HTTP responses).
│   │   ├── test_detailendpoint.py
│   │   ├── test_endpoint.py
│   │   ├── test_endpoint_strict_filter.py
│   │   ├── test_extras.py
│   │   ├── test_file_upload.py
│   │   ├── test_mapper.py
│   │   ├── test_multiformat_endpoint.py
│   │   ├── test_query.py
│   │   ├── test_request.py
│   │   └── test_response.py
│   └── integration/             — Integration tests against live NetBox in Docker.
│       ├── conftest.py          — Docker setup: spin up netbox-docker, wait for ready.
│       ├── test_circuits.py
│       ├── test_dcim.py
│       └── test_ipam.py
├── docs/                        — mkdocs site (API reference + guides).
├── .github/workflows/
│   ├── py3.yml                  — Lint + tests on every push/PR (matrix: Python × NetBox).
│   ├── publish.yml              — PyPI publish on GitHub release.
│   └── build-mkdocs.yml         — Deploy docs to GitHub Pages on push to master.
├── AGENTS.md                    — This file.
├── CLAUDE.md                    — Shim that pulls in this file.
├── CHANGELOG.md                 — Release history.
├── mkdocs.yml                   — Docs site config.
├── requirements.txt             — Runtime dependencies.
├── requirements-dev.txt         — Dev/test dependencies.
└── setup.py                     — Package metadata; version from setuptools_scm.
```

## Architecture

The library follows a layered architecture. Each layer wraps the one below it.

### API Layer (`pynetbox/core/api.py`)

`Api` is the main entry point. On construction it eagerly creates `App` instances for every built-in NetBox application (dcim, ipam, circuits, virtualization, extras, users, wireless, core, vpn, tenancy) plus a `PluginsApp` for plugin endpoints. It owns the `requests.Session`, authentication token, threading flag, and `strict_filters` setting.

### App Layer (`pynetbox/core/app.py`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [netbox-community/pynetbox](https://github.com/netbox-community/pynetbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
