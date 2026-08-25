---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

A bilingual (RU primary / EN secondary) Telegram bot that sells Telegram Stars over **two payment rails** — the **WATA Digital Goods API** (card/SBP) and **GreenGamePay / GGP** (crypto, TON) — with a referral program, partner-owned hosted bots, and admin-moderated withdrawals. Python 3.12, aiogram 3.x, aiohttp, SQLAlchemy 2 async, PostgreSQL, Redis, Alembic.

The two rails are independent and each is optional, auto-enabled from its own credentials: the card rail is on when `WATA_TOKEN` is set (`settings.wata_enabled`), the crypto rail when the GGP credentials + `GGP_TON_ADDRESS` are all set (`settings.crypto_enabled`). A `model_validator` requires at least one. There is no separate on/off or "crypto-only" flag.

## Commands

A local `.venv` exists. On this Windows host use the Bash tool with the venv interpreter directly:

```bash
.venv/Scripts/python.exe -m ruff check .          # lint (run before committing)
.venv/Scripts/python.exe -m ruff check --fix .    # autofix (import sort etc.)
.venv/Scripts/python.exe -m pytest -q             # full test suite
.venv/Scripts/python.exe -m pytest tests/test_wata_parsing.py::test_message_key_mapping  # single test
.venv/Scripts/python.exe -m mypy app              # type check
```

Docker (production / integration — cannot run in this dev environment, no Docker here):

```bash
docker compose up -d --build      # bot + postgres + redis; entrypoint runs `alembic upgrade head` first
docker compose logs -f bot
```

Alembic migrations auto-apply on container start (`entrypoint.sh`). To add one, create
`alembic/versions/000N_<slug>.py` with `down_revision` pointing at the previous head
(the chain is linear, `0001_initial` → … → the current head in `alembic/versions/`; `0007_crypto_orders`
added the `provider`/`ton_amount`/`ggp_tx_id` columns for the crypto rail).

### Dependency / Python-version caveat

`requirements.txt` pins versions for **Python 3.12** (the Docker image). This dev host only has
Python 3.14, for which the pinned `aiohttp`/`asyncpg`/`pydantic-core` have no wheels, so the local
`.venv` was created with **unpinned latest** versions instead. Code therefore must stay compatible
with both — it currently runs on aiogram 3.29 locally and the pinned 3.29 in Docker. Avoid APIs that
exist only in one minor version.

## Architecture

### One process, one aiohttp server (webhook mode only)

`app/main.py` is the entrypoint. The bot does **not** long-poll. A single aiohttp app (built in
`app/webhook/server.py`, extended in `main.py`) serves every inbound path behind a user-managed TLS
reverse proxy:

- `TELEGRAM_WEBHOOK_PATH` (default `/tg/webhook`) — main bot updates via aiogram `SimpleRequestHandler`
- `WEBHOOK_PATH` (default `/wata/webhook`) — WATA payment notifications (custom handler)
- `/partner/{bot_token}` — all partner bots, via aiogram `TokenBasedRequestHandler` (multibot)
- `/health`

`WEBHOOK_HOST` is **required** or the process exits. On startup the bot calls `set_webhook` for itself
and for every active partner bot; on shutdown it deletes its webhook. The Telegram secret token is
`TELEGRAM_WEBHOOK_SECRET` or, if empty, `sha256(BOT_TOKEN)`.

### Layering

`handlers (aiogram) → services → repositories → db.session` plus `wata`/`ggp` client packages and pure
`pricing`/`i18n` leaf modules.

- **`app/services.py`** is the core. Three service classes, each holding `Settings` + `Database`:
  - `OrderService` — pricing/quote, `create_order` (WATA) and `create_crypto_order` (GGP), `sync_order`, crypto delivery (`oldest_pending_crypto`, `try_deliver_crypto`, `deliver_ggp_order`), stats, language cache, `simulate_payment` (test mode). Holds an optional `GgpClient` (None when crypto disabled); `wata_enabled`/`crypto_enabled` gate the rails.
  - `ReferralService` — referral attribution, `credit_for_order` (idempotent per order), balance, withdrawals.
  - `PartnerService` — partner bot CRUD, per-bot markup, markup cache.
- **`app/db/repositories.py`** — thin per-aggregate repositories; each takes an `AsyncSession`. Upserts use Postgres `ON CONFLICT` (`pg_insert`).
- **`app/db/session.py`** — `Database.session()` async context manager commits on success, rolls back on error. **`expire_on_commit=False`**, so returning ORM objects from a service and reading their attributes after the session closes is safe (the codebase relies on this).

### Dependency injection into handlers

`main.py` puts shared singletons on the Dispatcher (`dp["settings"]`, `dp["service"]`, `dp["referral"]`,
`dp["partner"]`, and `dp["main_bot"]`); aiogram injects them into handlers by matching the parameter name.
`LanguageMiddleware` (`app/bot/middleware.py`) injects `lang: str` into every message/callback handler by
resolving the user's stored/Telegram language. The same `dp` serves the main bot **and** all partner bots,
so handlers must be bot-agnostic — use `event.bot.id` to tell which bot is running (e.g. to look up a
partner markup). `dp["main_bot"]` is that shared main bot: a handler firing on a *partner* bot uses it to
send notifications through the main bot (e.g. `cb_crypto_paid` → `notify_order_outcome`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [evansvl/telegram-stars-bot](https://github.com/evansvl/telegram-stars-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
