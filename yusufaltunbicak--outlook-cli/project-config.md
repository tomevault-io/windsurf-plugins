---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## What This Is

A Python CLI tool for Outlook 365 that uses OWA bearer token authentication via Playwright browser interception — no Azure app registration, admin consent, or API keys required. Entry point: `outlook` command.

## Build & Run

```sh
pip install -e .              # editable install (hatchling build system)
playwright install chromium   # required for auth
outlook login                 # first-time: opens browser, captures OWA bearer token
outlook inbox                 # verify it works
```

```sh
pytest               # run the full test suite
pytest -m smoke      # run only smoke tests (require live token)
```

## Architecture

### Two API layers

1. **Outlook REST v2** (`outlook.office.com/api/v2.0/me`) — standard mail, calendar, contacts, folders, per-message categories. Used by `OutlookClient` in `client.py`.
2. **OWA service.svc** (`outlook.cloud.microsoft/owa/service.svc`) — reverse-engineered endpoint for master category list operations (create/delete/rename/recolor) and message pinning (`UpdateItem` with `RenewTime`). Uses a non-standard pattern: JSON payload goes in the `x-owa-urlpostdata` header, body is empty. Used by `category_manager.py` and `client.py` (`pin_message`).

### Module responsibilities

- **`cli.py`** — Click group definition + command registration hub only (~92 lines). Imports from `commands/` modules.
- **`account.py`** — Account profile registry, active-profile resolution, path derivation, mailbox binding checks, and per-profile config loading.
- **`commands/`** — All CLI commands split into modules:
  - `_common.py` — shared helpers: active-account resolution, runtime config proxy, per-profile `_get_client`, `_handle_api_error`, `_wants_json`
  - `account.py` — `account add/list/current/switch/remove`
  - `auth.py` — `login`, `whoami`
  - `mail.py` — `inbox`, `read`, `thread`, `send`, `draft`, `draft-send`, `reply`, `reply-draft`, `forward`
  - `schedule.py` — `schedule`, `schedule-list`, `schedule-cancel`, `schedule-draft`
  - `search.py` — `search`
  - `folders.py` — `folders`, `folder`
  - `categories.py` — `categories`, `categorize`, `uncategorize`, `category-create/rename/clear/delete`
  - `signatures.py` — `signature-pull`, `signature-list`, `signature-show`, `signature-delete`
  - `manage.py` — `mark-read`, `move`, `delete`, `flag`, `pin`
  - `attachments.py` — `attachments`
  - `calendar.py` — `calendar`, `event`, `event-create/update/delete/instances/respond`, `calendars`, `free-busy`, `people-search`
  - `contacts.py` — `contacts`
- **`exceptions.py`** — Structured exception hierarchy: `OutlookCliError` → `TokenExpiredError`, `RateLimitError`, `ResourceNotFoundError`, `AuthRequiredError`, `AccountError`. Includes `error_code_for_exception()` mapping.
- **`client.py`** — `OutlookClient` wraps httpx for REST v2 API. Manages per-profile display-number-to-real-ID mapping (short `#1, #2` numbers → long Outlook IDs). Handles rate limiting (429 retry) and token expiry (401). `get_thread()` fetches conversation chains.
- **`auth.py`** — Playwright-based token capture. Intercepts bearer tokens from OWA network requests. Picks the best token by testing against multiple endpoints. Enforces strict mailbox binding per account profile and stores token + browser SSO state in profile-scoped paths. Also supports `--with-token` for direct token input (skips browser, validates JWT format and mailbox binding).
- **`category_manager.py`** — Standalone module for OWA master category operations. Has its own `_owa_request` helper (separate from `client.py`'s `_owa_action`). `rename_category` and `clear_category` do bulk message propagation via REST v2.
- **`signature_manager.py`** — Signature management: pull from SentItems, save as HTML files in the selected profile's signature directory, append to outgoing emails. Handles plain text → HTML conversion when signature is used.
- **`models.py`** — Dataclasses (`Email`, `Folder`, `Attachment`, `Event`, `Attendee`, `Contact`, `EmailAddress`) with `from_api()` class methods that parse Outlook REST v2 JSON. `Email` includes `categories: list[str]`, `flag_status` ("notFlagged"/"flagged"/"complete"), `flag_due: datetime | None`. `Event` includes `attendees: list[Attendee]`, `recurrence`, `event_type` (SingleInstance/Occurrence/Exception/SeriesMaster), `series_master_id`, `display_num`.
- **`formatter.py`** — Rich table output. `Console(stderr=True)` so JSON piping stays clean on stdout. `print_thread()` for conversation view. Inbox flags column shows `*` (unread), `@` (attachment), `!` (flagged), `v` (flag complete). Email detail view shows flag status with due date.
- **`serialization.py`** — `to_json_envelope()` wraps data in `{ok, schema_version, data}` for stdout. `error_json()` for structured errors. `to_json()` / `save_json()` for raw file export. Accepts optional `tz` parameter to convert datetimes to a target timezone (outputs single ISO 8601 string with offset).
- **`config.py`** — Global YAML config loader with deep-merge defaults; per-profile config overlays are resolved via `account.py`.
- **`constants.py`** — URLs and root cache/config paths.

### Key patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yusufaltunbicak/outlook-cli](https://github.com/yusufaltunbicak/outlook-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
