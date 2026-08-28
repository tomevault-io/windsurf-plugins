---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What rssFriends is

**A social, RSS-friendly radar for the open web** — not a reader. No unread counters, no backlog, no algorithm. Users *watch* feeds and people they trust; they don't clear a queue. Every URL is subscribable, every export is OPML, and the roadmap points at ActivityPub. When writing copy or naming things, prefer **radar / watch / catch / signal / feeds / friends** over **reader / read / inbox / articles / subscribers**. See `docs/design-language.md` for the full voice notes.

## Commands

**Package + venv management via `uv`.** All dev commands go through it.

```bash
uv sync --all-extras                        # install (creates .venv)
uv run uvicorn app.main:app --reload        # dev server → http://localhost:8000
uv run pytest -q                            # all tests
uv run pytest tests/test_public.py::test_public_radar_shows_categories_and_feeds -vv
uv run ruff check .                         # lint
uv run ruff check --fix .                   # autofix
```

Or the Makefile aliases: `make install / dev / test / lint / fmt / up / down`.

**Backup + restore** (WAL-safe via `sqlite3 .backup`):
```bash
make backup
make restore FROM=backups/rssfriends-<ts>.sqlite.gz
```

**Version bump** — edit `app/__init__.py` (`__version__`) *and* `pyproject.toml`. The version is baked into static asset URLs (`?v={version}`) so bumps also bust Cloudflare's edge cache.

## Deployment (production)

Two compose files:
- `docker-compose.yml` — plain dev.
- `docker-compose.prod.yml` — with a Caddy sidecar for TLS.
- On the shared `pathseeker` host we use `docker-compose.pathseeker.yml` (bound to `127.0.0.1:8300`, TLS handled by the host's nginx; that file is NOT in git). Deploy via `make deploy` — runs tests + lint, `rsync`s to `root@pathseeker:/srv/rssfriends/`, rebuilds the container, and verifies `/version`. Use `make deploy-check` for a dry-run. See `docs/deploy.md` for the manual/first-time setup. **Never call `rsync --delete` by hand** — the Makefile's exclude list has trip-wire history (once wiped prod `.env`).

## Architecture

FastAPI + HTMX + Jinja2 + SQLite (WAL) + APScheduler, one Docker container, no JS build step.

### Data model (`app/db.py`)

Eight tables. Every join is one hop; every FK is an integer.

- **`User`** — magic-link users. Fields: `email`, `handle`, `visibility` ∈ `{public, unlisted, private}`, `activity_secret` (per-user HMAC key for signing private activity.rss URLs).
- **`MagicToken`** — single-use auth tokens with a short TTL.
- **`Category`** — user-scoped; the label a user groups feeds under.
- **`Feed`** — **global** row deduped by URL. Every user subscribed to the same URL shares this row. Has `favicon_url`, `last_fetched_at`, `consecutive_failures`, `last_error`.
- **`Subscription`** — a (user, feed, category) triple. Unique per (user, feed).
- **`FeedItem`** — items parsed from a Feed. Dedup by `(feed_id, guid)`. `title` can be derived from summary if the source omits `<title>` (Mastodon).
- **`Favorite`** — composite PK `(user_id, feed_item_id)` — physically deduped by the DB, not by app logic.
- **`Follow`** — composite PK `(follower_id, followee_id)`. `approved_at IS NULL` = pending request (private followees only); non-NULL = approved.
- **`Activity`** — append-only log. `kind` ∈ `{subscribed, favorited, followed, forked, created_category}`. `subject` / `subject_url` are **denormalized** so the timeline still renders after the referenced row is deleted.
- **`WallabagAccount`** — one-per-user OAuth link to a self-hosted [Wallabag](https://wallabag.org). Bookmarking an item fires a background POST to `/api/entries.json`. Wallabag only supports the `password` grant, so we store the user's Wallabag username + password too. Sensitive fields (`password`, `client_secret`, `access_token`, `refresh_token`) are **encrypted at rest** — see below.

**Never write raw values into `Feed.favicon_url` / `Feed.title` / `Feed.url` without going through the helpers** — see below.

### Secrets & field encryption (`app/crypto.py`)

**One env var, `RSSFRIENDS_SESSION_SECRET`, does three unrelated jobs.** Rotating it invalidates all three at once — that's intentional but load-bearing:

1. Signs Starlette session cookies (`app/main.py`) — rotation logs everyone out.
2. Mixed with `User.activity_secret` when signing `activity.rss` URLs on private radars.
3. Via HKDF-SHA256 (salt `"rssfriends/field-encryption/v1"`) derives a 32-byte Fernet key that encrypts sensitive `WallabagAccount` fields at rest (`app/crypto.py`, `encrypt()`/`decrypt()`). Rotation makes every stored Wallabag credential unrecoverable — `decrypt()` returns `""` on `InvalidToken` so callers can re-prompt safely instead of leaking plaintext.

Threat model for field encryption: a stolen DB dump alone is opaque (backup theft, cloud snapshot). A full server compromise still exposes creds, since the running app must be able to decrypt what it encrypted — that's inherent, not a bug.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pesarkhobeee/FediverseRssFriends](https://github.com/pesarkhobeee/FediverseRssFriends) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
