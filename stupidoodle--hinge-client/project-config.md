---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) working in this repository.

## Project Overview

`hinge-client` — a reverse-engineered, typed **async** client for the Hinge dating-app API.
Pure library: auth, recommendations/standouts, profiles, rating (like/skip/note), preferences,
content, prompts, and Sendbird chat over `httpx` + `websockets`, with pydantic models. Targets
parity with the iOS Hinge app (version `9.82.0`, build `11616`).

This is a **library only** — no web server, no database, no background services. State is the
caller's concern; the client persists only a small per-phone session file (path configurable via
`HINGE_SESSIONS_DIR`).

Distribution name **`hinge-client`**; import as `import hinge`.

## Commands

Always use `uv run` (never bare `python`/`python3`). **NEVER** hand-edit `pyproject.toml`
dependency entries — use uv (`uv add <pkg>`, `uv add --group dev <pkg>`, `uv remove <pkg>`).

```bash
uv sync                            # install deps
uv run pre-commit install          # install git hooks
make setup                         # deps + hooks

uv run ruff check --fix            # lint (autofix)
uv run ruff format                 # format
uv run ty check src/               # type-check — Astral's `ty` (NOT mypy)
make check                         # lint + format-check + ty + leak-canary

uv run pytest -m "not integration" --cov=src/hinge --cov-branch --cov-fail-under=100
uv run pytest tests/path::TestClass::test_name      # single test
make build                         # sdist+wheel + twine check + contents audit
```

Type checker is **`ty`** (config under `[tool.ty]`); there is no mypy. Lint + format are **ruff**.

## Architecture

Single flat package under `src/hinge/`:

- `client.py` — the async `HingeClient` (httpx + websockets). Auth state machine, JWT
  auto-refresh, session persistence, all REST endpoints, and the Sendbird chat surface.
- `models.py` — pydantic schemas for API requests/responses (camelCase⇄snake_case aliases).
- `enums.py` — **protocol** enums only (`RatingAction`, `RatingInitiatedWith`, `FeedOrigin`,
  `ContentType`, `GenderEnum`). Display-catalogue values are NOT enums here — see Catalog.
- `error.py` — exception types (`HingeAuthError`, `HingeEmail2FAError`, …).
- `data/catalog.json` — bundled static enum-label data (see Data note).
- `core/logging.py` — standalone structlog setup (opt-in pretty console; see Logging).
- `core/catalog.py` — in-memory `Catalog` (`EnumCatalog` + `PromptsManager`), no DB.

### Catalog (in-memory, no DB)
- **Prompts** are fetched live (`POST /prompts`) and cached in memory (opt-in disk JSON).
- **Enum labels** (religion, ethnicity, language, dating intention, …) are not served by any
  endpoint, so they ship as bundled `data/catalog.json` and are loaded in memory. The live
  `/config/v3` call refreshes only the *valid-ID set*. Special values: `-1` Open-to-All,
  `0` Prefer-not-to-say, `99999999` Unknown.

### Logging
structlog throughout. The library does **not** configure global logging on import. Callers opt in
to formatted/colored console output via `hinge.core.logging.configure_logging(...)`. Never log
tokens, JWTs, or OTPs.

## Hinge API

Base URL `https://prod-api.hingeaws.net`. Bearer JWT + UUID device/install/session headers; no
request signing. Chat is delegated to Sendbird (app id `3CDAD91C-1E0D-4A0D-BBEE-9671988BF9E9`):
REST `https://api-{app_id_lower}.sendbird.com`, WS `wss://ws-{app_id_lower}.sendbird.com`.

| Area | Method | Endpoint |
|---|---|---|
| Install id | POST | `/identity/install` |
| Request OTP | POST | `/auth/sms/v2/initiate` |
| Submit OTP | POST | `/auth/sms/v2` |
| Device/email validate | POST | `/auth/device/validate` |
| Refresh token | POST | `/auth/refresh` |
| Recommendations | POST | `/rec/v2` |
| Recycle feed | POST | `/user/repeat` |
| Standouts | POST | `/standouts/v2`, `/standouts/v3` |
| Likes-you | POST | `/like/v2` |
| Like limit | GET | `/likelimit` |
| Self profile | GET | `/user/v2`, `/user/v3` |
| Public profile | POST | `/user/v2/public`, `/user/v3/public` |
| Traits | POST | `/user/v2/traits` |
| Profile state | GET | `/profilestate/profile`, `/profilestate/basics/missing` |
| Rate (like/skip/note) | POST | `/rate/v2/initiate`, `/rate/v2/respond`, `/rate/v2/match` |
| Preferences | GET/PATCH | `/preference/v2/selected` |
| Content | POST | `/content/v2`, `/content/v2/public`, `/content/v1/photos`, `/content/v1/answers`, `/content/v1/answer/evaluate`, `/content/v1/settings` |
| Enum/preference config | GET | `/config/v3` |
| Prompts library | POST | `/prompts` |
| Store / boost / moderation | POST/GET | `/store/v2/account`, `/boost/status`, `/flag/textreview` |

(Authoritative list lives in `client.py`.)

## Data note

`src/hinge/data/catalog.json` is **bundled generated data** — treat it as a build artifact, do not
hand-edit. Keep any details of how that data was produced **out of tracked files**; a leak-canary
test (`tests/test_no_secrets.py`) fails the build if such details appear in the repo.

## Testing

- Unit/property/contract tests run with no network (`respx` for httpx, `AsyncMock` for websockets,
  `hypothesis` for property tests). 100% branch coverage is enforced.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Stupidoodle/hinge-client](https://github.com/Stupidoodle/hinge-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
