---
trigger: always_on
description: Provides book list fetching. `SearchCataloger` for keyword search, `FollowedCataloger` for followed books. Contains HTML parsing utilities.
---

# AGENT.md — kmoe-manga-downloader

## Project Overview

`kmdr` (Kmoe Manga Downloader) is a Python CLI application for downloading manga from the [Kmoe](https://kxx.moe/) website. It provides login, download (with multi-part/resume/failover), credential pool management, and persistent configuration — all through a rich terminal interface.

- **Entry point**: `kmdr.main:entry_point` (registered as `kmdr` CLI command)
- **Python**: ≥ 3.9
- **Package layout**: `src/kmdr/` (src layout)
- **Build**: Poetry + `poetry-dynamic-versioning` (PEP 517 backend)
- **Versioning**: Dynamic from Git tags via `poetry-dynamic-versioning`, placeholder in `src/kmdr/_version.py`

## Architecture

### Core (`src/kmdr/core/`)

The core layer defines **abstract base classes**, a **Registry-based plugin system**, data models, and shared infrastructure.

#### Registry Pattern (`registry.py`)

The central design pattern. `Registry[T]` is a generic container that maps command-line arguments (`argparse.Namespace`) to module implementations via predicate matching.

```
Registry.register(hasattrs, containattrs, hasvalues, predicate, order)
Registry.get(args) → T  # resolves and instantiates the matching implementation
```

Matching logic: `{predicate} OR {hasvalues} AND ({hasattrs} OR {containattrs})`. Registrations are sorted by `order` (lower = higher priority).

Eight global registries are defined in `bases.py`:

| Registry           | Base Class       | Purpose                    |
|--------------------|------------------|----------------------------|
| `SESSION_MANAGER`  | `SessionManager` | HTTP session lifecycle     |
| `AUTHENTICATOR`    | `Authenticator`  | Login / cookie auth        |
| `CATALOGERS`       | `Cataloger`      | Fetch book list collections|
| `LISTERS`          | `Lister`         | Fetch book & volume info   |
| `PICKERS`          | `Picker`         | Filter/select volumes      |
| `DOWNLOADER`       | `Downloader`     | Download volumes           |
| `CONFIGURER`       | `Configurer`     | Config operations          |
| `POOL_MANAGER`     | `PoolManager`    | Credential pool management |

#### Mixin Contexts (`context.py`)

Base classes use cooperative multiple inheritance with context mixins:

- `TerminalContext` — provides `_console` (Rich Console) and `_progress` (Progress bar)
- `SessionContext` — provides `_session` and `_base_url` via `ContextVar`
- `ConfigContext` — provides `_configurer` (singleton JSON config manager)
- `CredentialPoolContext` — provides `_pool` (lazy-loaded `CredentialPool`)

#### Data Models (`structure.py`)

- `BookInfo` — immutable book metadata (id, name, author, url, status)
- `VolInfo` — immutable volume metadata (id, name, type, pages, size)
- `Credential` — mutable user credential (username, cookies, quotas, status, order)
- `QuotaInfo` — user/VIP quota tracking with unsynced usage
- `Config` — persistent config (options, cookie, base_url, cred_pool)
- `VolumeType` — enum: `VOLUME`, `EXTRA`, `SERIALIZED`

#### Key Modules

- **`defaults.py`** — `argparse` definitions, `Configurer` singleton (reads/writes `~/.kmdr` JSON), argument merging for persistent options
- **`session.py`** — `KmdrSessionManager`: creates `aiohttp.ClientSession`, probes mirror URLs with priority sorting
- **`pool.py`** — `CredentialPool` (tiered round-robin scheduling with sticky-preferred cred) and `PooledCredential` (quota reservation with transactional commit/rollback)
- **`utils.py`** — `async_retry` decorator (exponential backoff, redirect handling), `PrioritySorter`, `SharedAwaitable` (for concurrent auth+catalog), callback construction, UA rotation
- **`console.py`** — `info()` / `debug()` / `log()` output functions, adapts between interactive (`print`) and non-interactive (`log`) modes; **Tool Call Mode**: `emit()` for final result, `emit_progress()` for real-time NDJSON progress
- **`encoder.py`** — `KmdrJSONEncoder` (standard JSON encoder for dataclasses) and `SafeJSONEncoder` (auto-desensitizes fields marked `sensitive=True`)
- **`error.py`** — Exception hierarchy rooted at `KmdrError(RuntimeError)` with two-digit status codes; includes solution suggestions
- **`patch.py`** — Monkey-patches `Console.status()` to support nested/stacked status contexts in async scenarios

#### Error Status Codes (`error.py`)

Two-digit status codes categorized by domain: 0 (success), 1x (init/args), 2x (auth/cred), 3x (redirect), 4x (input), 5x (server/network). See `error.py` for details.

#### Tool Call Mode

When invoked with `--mode toolcall`, kmdr outputs structured NDJSON for AI agent consumption:

- **Output format**: Each line is a JSON object; final line is `{"type": "result", "code": N, "msg": "...", "data": {...}}`
- **Progress updates**: Download commands emit `{"type": "progress", ...}` lines during execution
- **Desensitization**: `SafeJSONEncoder` automatically replaces sensitive fields (cookies, passwords) with `"***SENSITIVE***"`
- **Fast Auth**: `--fast-auth` flag skips network validation, uses local credential pool directly via `LocalPoolAuthenticator`

### Modules (`src/kmdr/module/`)

Concrete implementations registered to the core registries. Each subdirectory is a feature module.

#### `cataloger/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrisis58/kmoe-manga-downloader](https://github.com/chrisis58/kmoe-manga-downloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
