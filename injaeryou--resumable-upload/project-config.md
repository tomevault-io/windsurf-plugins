---
trigger: always_on
description: This file describes how Claude Code (and other AI coding agents) should work in this repository. Human contributors: treat this as a living cheat-sheet too.
---

# CLAUDE.md — Project Conventions for AI-driven Development

This file describes how Claude Code (and other AI coding agents) should work in this repository. Human contributors: treat this as a living cheat-sheet too.

## Project Overview

`resumable-upload` is a Python implementation of the [TUS resumable upload protocol v1.0.0](https://tus.io/protocols/resumable-upload.html). It ships both server and client components with **zero runtime dependencies** for the core path. Cloud storage backends (S3 / GCS / Azure) are opt-in via extras.

- **Status**: published on PyPI — `pip install resumable-upload`
- **Python**: 3.9 through 3.14 (keep 3.9 floor)
- **License**: MIT
- **Compliance matrix**: see `TUS_COMPLIANCE.md`

## Architecture (30-second tour)

```
resumable_upload/
├── __init__.py            — public exports (keep surface minimal)
├── exceptions.py          — TusHookError, TusCommunicationError, TusUploadFailed
├── checksum.py            — multi-algorithm checksum registry
├── fingerprint.py         — default SHA-256 full-file fingerprint
├── metrics.py             — zero-dependency Prometheus counter registry
├── asgi.py                — TusASGIApp (FastAPI / Starlette adapter)
├── cli.py / __main__.py   — `resumable-upload` CLI: `serve` (server) + `upload`/`download`/`info` (client)
│
├── server/
│   ├── __init__.py        — re-exports TusServer, TusServerCore, TusHTTPRequestHandler
│   ├── core.py            — TusServerCore (full TUS 1.0.0 implementation; exposes handle_request + handle_request_async)
│   ├── server.py          — TusServer(TusServerCore), the canonical class to instantiate
│   └── http_handler.py    — TusHTTPRequestHandler (sync http.server glue)
│
├── client/
│   ├── __init__.py        — re-exports TusClient, Uploader, UploadStats, AsyncTusClient
│   ├── client.py          — TusClient (mixes the three mixins below)
│   ├── _mixin_base.py     — _ClientAttrs: shared attribute / method shape used by mixins
│   ├── _protocol.py       — pure transport-agnostic helpers shared by sync + async clients
│   ├── protocol.py        — ProtocolMixin: encode_metadata, get_metadata, get_upload_info, get_server_info
│   ├── concatenation.py   — ConcatenationMixin: create_partial_upload, create_final_upload
│   ├── parallel.py        — ParallelUploadMixin: _upload_parallel
│   ├── uploader.py        — Uploader (low-level chunk control)
│   ├── stats.py           — UploadStats
│   └── aio/               — async client package (httpx, requires [async] extra)
│       ├── __init__.py    — re-exports AsyncTusClient, AsyncUploader
│       ├── _http.py       — lazy httpx import + request/error mapping
│       ├── client.py      — AsyncTusClient (async equivalent of TusClient)
│       └── uploader.py    — AsyncUploader (async chunk control)
│
├── storage/
│   ├── __init__.py        — re-exports Storage, SQLiteStorage, S3/GCS/Azure (lazy)
│   ├── base.py            — Storage ABC; sync interface + *_async surface (defaults to asyncio.to_thread)
│   ├── sqlite_storage.py  — SQLiteStorage (default)
│   ├── s3_storage.py      — S3 backend (optional, boto3)
│   ├── gcs_storage.py     — GCS backend (optional, google-cloud-storage)
│   └── azure_storage.py   — Azure backend (optional, azure-storage-blob)
│
├── url_storage/
│   ├── __init__.py        — re-exports URLStorage + 3 implementations
│   ├── base.py            — URLStorage ABC
│   ├── memory_url_storage.py
│   ├── sqlite_url_storage.py
│   └── file_url_storage.py
│
└── locks/
    ├── __init__.py        — re-exports LockBackend + implementations
    ├── base.py            — LockBackend ABC
    ├── memory_lock.py     — InMemoryLockBackend (default)
    └── redis_lock.py      — RedisLockBackend (optional, redis)

tests/                     — pytest, one test file per module
examples/                  — runnable Flask/FastAPI/Django integration examples
docs/                      — user-facing mkdocs site (do not repurpose)
.docs/                     — AI-only artifacts (gitignored) — see below
```

Legacy import paths are kept alive but emit ``DeprecationWarning`` on
first use, **deprecated in 0.0.6, scheduled for removal after 0.1.2**:
``resumable_upload.storage_s3``, ``storage_gcs``, ``storage_azure``,
``locks_redis``, and ``client.base`` are ``sys.modules`` aliases for the
new submodules. New code (and the project's own tests / examples / docs)
should import directly from the new packages; the aliases exist only to
give downstream users one release cycle to migrate.

## Tech Stack & Constraints

- Runtime: **stdlib only** for `resumable_upload.server`, `resumable_upload.client`, `resumable_upload.storage` (SQLite is stdlib).
- Never add a runtime dependency to the core without explicit user approval.
- Cloud backends use their official SDK (`boto3`, `google-cloud-storage`, `azure-storage-blob`) gated by `[project.optional-dependencies]` extras.
- Type checker: **ty** (Astral). Never reintroduce mypy — see `14798d2`.
- Linter/formatter: **ruff** (config in `pyproject.toml`). Line length 100.
- Test runner: **pytest** + **pytest-cov**. Fixtures live next to tests; no `conftest.py` sprawl.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [injaeryou/resumable-upload](https://github.com/injaeryou/resumable-upload) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
