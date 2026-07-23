---
trigger: always_on
description: Guidance for Claude Code (and other AI agents) working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (and other AI agents) working in this repository.
`AGENTS.md` and `GEMINI.md` are symlinks to this file — edit **CLAUDE.md only**; the others follow automatically.

---

## 1. What this project is

**umami-analytics** is a Python client SDK for [Umami](https://umami.is), the open-source,
privacy-preserving web analytics platform. It lets Python **backends** send custom events, page
views, and revenue transactions to a self-hosted or Umami Cloud instance, and query basic stats —
useful for tracking deep business-logic events (e.g. a course purchase) that have no natural
front-end HTML trigger.

- **PyPI package:** `umami-analytics` &nbsp;(`pip install umami-analytics` / `uv add umami-analytics`)
- **Import name:** `umami` &nbsp;(i.e. `import umami`)
- **Repo:** https://github.com/mikeckennedy/umami-python
- **License:** MIT © Michael Kennedy <michael@talkpython.fm>
- **Status:** Stable (1.0+), actively maintained; used in production at Talk Python Training.
- **Python:** 3.10+ (classifiers cover 3.10–3.15).
- **Runtime deps:** `httpx2` (an API-compatible fork of `httpx`), `pydantic`.
- **Scope is intentionally partial:** it wraps the most common Umami endpoints, not the whole API.
  Remaining endpoints are documented at https://umami.is/docs/api.

---

## 2. Repository layout — READ THIS FIRST

This is a **two-tier layout**. The repo root holds docs and tool config; the installable package
lives one level down in `umami/`, and the importable package is nested again at `umami/umami/`.
**Getting the working directory right matters** — see §4.

```
umami-python/                     ← REPO ROOT (docs + lint/type config)
├── CLAUDE.md  AGENTS.md  GEMINI.md   ← this file + two symlinks to it
├── README.md                     top-level docs
├── change-log.md                 Keep a Changelog format (UPDATE THIS for user-facing changes)
├── LICENSE                       MIT
├── ruff.toml                     ★ ruff config lives HERE (repo root)
├── pyrefly.toml                  ★ pyrefly config lives HERE (repo root)
├── .pre-commit-config.yaml       ruff --fix on commit
├── .github/FUNDING.yml           only GitHub file — NO CI workflows exist
├── dev-docs/                     design notes & package reference guides
├── plans/                        in-progress design/refinement plans
├── readme_resources/             README image assets
└── umami/                        ← PACKAGE DIR (build + test config)
    ├── pyproject.toml            ★ package metadata, deps, version, pytest config (hatchling)
    ├── requirements.txt          mirrors runtime deps (httpx2, pydantic)
    ├── tox.ini                   LEGACY / stale (py27–py37) — not the source of truth
    ├── README.md  LICENSE
    ├── umami/                    ← IMPORTABLE PACKAGE ("import umami")
    │   ├── __init__.py           public API re-exports + __all__
    │   ├── impl/__init__.py      ALL implementation + module-global state
    │   ├── models/__init__.py    Pydantic response models
    │   ├── errors/__init__.py    exception classes
    │   ├── urls.py               API endpoint path constants
    │   └── py.typed              PEP 561 marker — package ships type hints
    ├── tests/                    pytest suite (conftest.py + test_*.py)
    └── example_client/           runnable end-to-end demo (client.py + settings)
```

**The gotcha to remember:** `ruff.toml` and `pyrefly.toml` are at the **repo root**, but
`pyproject.toml` and the pytest config are in **`umami/`**. So linting/type-checking run from the
root, while tests run from `umami/`.

---

## 3. Architecture

- **Module-level function API, not a client class.** Everything is called as `umami.some_function(...)`.
- **Global state** lives in `umami/umami/impl/__init__.py` (`url_base`, `auth_token`,
  `default_website_id`, `default_hostname`, `tracking_enabled`, user-agent strings, `__version__`).
  Configuration functions mutate these globals.
- **`__init__.py` is a thin facade:** it re-exports the public functions from `.impl` and exposes the
  `models` and `errors` submodules. The full public surface is the `__all__` list there — keep it in
  sync when adding/removing public functions.
- **Sync + async parity:** nearly every operation has a sync function and an `_async` twin with an
  identical signature and return type (e.g. `new_event` / `new_event_async`). **Preserve this parity** —
  if you change one, change the other.
- **HTTP:** `import httpx2 as httpx`. Sync uses module-level `httpx.post`/`httpx.get`; async uses an
  `httpx.AsyncClient()` context manager. Auth is `Authorization: Bearer <token>`; non-2xx responses
  call `resp.raise_for_status()`.
- **Version** is read at runtime via `importlib.metadata.version('umami-analytics')` (fallback `'0.0.0'`).

### Public API at a glance (`umami.__all__`)

| Group | Functions |
|---|---|
| Config (sync only) | `set_url_base`, `set_website_id`, `set_hostname`, `enable`, `disable` |
| Auth | `login`/`login_async`, `is_logged_in`, `verify_token`/`verify_token_async` |
| Send events | `new_event`/`_async`, `new_revenue_event`/`_async`, `new_page_view`/`_async` |
| Query stats | `websites`/`_async`, `website_stats`/`_async`, `active_users`/`_async` |
| Health | `heartbeat`/`_async` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikeckennedy/umami-python](https://github.com/mikeckennedy/umami-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
