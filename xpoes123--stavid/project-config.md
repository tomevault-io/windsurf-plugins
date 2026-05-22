---
trigger: always_on
description: Discord bot for David and Stephanie managing apartment life and personal habit tracking.
---

# StavidBot — Claude Code Context

## Project Overview
Discord bot for David and Stephanie managing apartment life and personal habit tracking.
Named "Stavid" (Stephanie + David).

## Tech Stack
- **Python 3.12** (see `.python-version`)
- **discord.py 2.5+** — slash commands via `app_commands`
- **SQLAlchemy 2.0 async** + **asyncpg** — all DB ops are async
- **Alembic** — migrations (autogenerate-friendly)
- **PostgreSQL** — local on the same VPS as the bot
- **FastAPI + uvicorn** — local-only HTTP API for Sage (see "Sage integration" below)

## Deployment

Stavid runs on the Hetzner VPS at `87.99.136.82` under `/opt/stavid` as a
systemd service named `stavid`. There is no Heroku, no `Procfile`, no
release phase. Deploys are autonomous: when a PR is merged on GitHub,
[Sentinel](../sentinel/) (the engineering bot, also on the same VPS) clones
the repo, copies files into `/opt/stavid` (preserving `.env` and `venv`),
runs `pip install -e .`, smoke-tests imports, and `systemctl restart stavid`.

Manual deploys / inspection from the VPS shell:

```bash
cd /opt/stavid
sudo systemctl restart stavid
sudo journalctl -u stavid -f
venv/bin/alembic current
venv/bin/alembic upgrade head
```

Migrations are auto-applied on every systemd restart via a drop-in
override at `deploy/stavid.service.d/30-alembic.conf` — it adds an
`ExecStartPre=alembic upgrade head` so a Sentinel auto-deploy of a
PR with a new migration applies cleanly without anyone SSHing in.

If the upgrade fails the bot won't start — check `journalctl -u
stavid` and run `alembic` by hand to diagnose. One-time install on
a fresh box:

```bash
sudo install -m 0644 -D \
  /opt/stavid/deploy/stavid.service.d/30-alembic.conf \
  /etc/systemd/system/stavid.service.d/30-alembic.conf
sudo systemctl daemon-reload
sudo systemctl restart stavid
```

## Project Structure
```
src/
  main.py        — Bot entrypoint; auto-loads all cogs in src/cogs/ recursively
  db.py          — SQLAlchemy engine, Base, all ORM models
  utils.py       — Shared helpers (resolve_partner, user IDs)
  api.py         — Sage-facing local HTTP API (FastAPI). Bound to 127.0.0.1.
  cogs/
    basic.py        — /help, /wifi
    budget.py       — /venmo, /pay, /rent, /wifi_bill, /ledger
    reminders.py    — /remind, /reminders, /remove_reminder, /reset_reminders + 60s firing loop
    playoff.py      — Habit tracker: daily check-ins, individual W/L, weekly review
    bucket.py       — Bucket list tracking
    datenight.py    — Date night logging
    outings.py      — Outings/activity wishlist with weighted roulette
    shopping.py     — Shopping list with Amazon OG scraping
    supplies.py     — Household supplies tracking
    watchlist.py    — Watchlist (movies/shows)
    chores.py       — Recurring chore templates + instances + materialize loop
    inbox/          — Channel-as-inbox routing (one cog per channel)
migrations/
  versions/    — Alembic migration files
```

## Key Conventions

### Adding a New Feature
1. Add the ORM model to [src/db.py](src/db.py) (under the `Base` class)
2. Run `alembic revision --autogenerate -m "description"` to generate a migration
3. Create or edit a cog in `src/cogs/` — the bot auto-loads everything in that package recursively
4. Cogs must end with `async def setup(bot): await bot.add_cog(YourCog(bot))`

### Database Sessions
Always use `async with self.bot.db() as s:` — never create your own engine.

### Partner Resolution
Use `resolve_partner(interaction)` from `src/utils.py` to get the other user. Relies on `PARTNER_IDS` env var.

### Background Tasks
Use `discord.ext.tasks` loop decorators inside a cog. Start them in `__init__` and cancel in `cog_unload`. Wrap the loop body in `before_loop` that awaits `bot.wait_until_ready()`.

### Channel-as-Inbox
Channels in the apartment guild can be wired up so plain messages turn into
domain rows. Each routing cog lives in `src/cogs/inbox/<channel>.py`,
filters by channel name, parses the message, writes the row, and reacts
✅ on success or ⚠️ on parse failure. Adding a new channel means adding
one file under that directory.

## Sage integration

Sage runs on the same VPS as Stavid. Stavid exposes a small read/write HTTP
API for Sage at **`127.0.0.1:7780`** (see [src/api.py](src/api.py)). The
flow is one-directional — **Sage calls Stavid; Stavid never calls Sage**.

- Bound to loopback only; no external traffic possible.
- Bearer-token auth via `STAVID_API_TOKEN` env var. Sage holds the same token.
- Endpoints (all scoped to the apartment guild):
  - `GET /healthz`
  - `GET /shopping`, `POST /shopping`
  - `GET /watchlist`, `POST /watchlist`
  - `GET /bucket`, `POST /bucket`
  - `GET /outings`, `POST /outings`
  - `GET /reminders`
  - `GET /ledger`
  - `GET /summary` — aggregate digest for Sage's morning check-in

This contract is stable. Treat `src/api.py` as load-bearing for Sage —
only modify additively, and never break existing response shapes without
also updating Sage.

## Environment Variables
| Variable | Description |
|----------|-------------|
| `DISCORD_TOKEN` | Bot token from Discord Developer Portal |
| `DATABASE_URL` | PostgreSQL connection string |
| `PARTNER_IDS` | Comma-separated Discord user IDs for David and Stephanie |
| `wifi_name` | Guest WiFi SSID |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xpoes123/stavid](https://github.com/xpoes123/stavid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
