---
trigger: always_on
description: Guidance for Claude Code working in this repo. Subdirectory `CLAUDE.md` files cover layer-specific rules.
---

# CLAUDE.md

Guidance for Claude Code working in this repo. Subdirectory `CLAUDE.md` files cover layer-specific rules.

## What this is

Per-ticker options analytics, watchlist-driven. Three processes share a single Postgres:

- **Next.js 16 web** (`web/`, port 3001) — Argon dark theme, RSC for landing pages, client islands for tabs
- **FastAPI** (`src/uw_scan/api/`, port 8400) — read-only over the warm store, mutations only via `/jobs`
- **APScheduler worker** (`src/uw_scan/worker/`) — full-scan / OHLC / spot-refresh / rescan-poll / nightly vol rollup

Postgres schema `uw_scan`, owned by role `argon_app` (NOSUPERUSER). UW (Unusual Whales) is the primary data source; xenon's IB realtime WS is the primary intraday spot feed (massive WS is the automatic fallback); massive.com supplies daily OHLC. **Never fall back to Yahoo.**

**Three-tier DB isolation** — `uw_scan.config._enforce_db_isolation` refuses to start on a `(host, db_name)` mismatch (override with `UW_SCAN_ALLOW_DB_MISMATCH=1` for one-off scripts):

| Host | DB name | Writer | Reset |
|------|---------|--------|-------|
| `100.66.147.98` (Mac mini, Tailscale) | `option_wizard` | macmini launchd stack only | persistent (prodlike) |
| `127.0.0.1` (MacBook / CI) | `option_wizard_local` | local `bash scripts/dev.sh` | persistent (dev-owned) |
| either host | `option_wizard_test` | `uv run pytest` | wiped per-fixture (DROP SCHEMA CASCADE) |

MacBook runs fully local by default. To point at the mini for a browse session, `.env.local` must override BOTH `UW_SCAN_DB_HOST=100.66.147.98` AND `UW_SCAN_DB_NAME=option_wizard` (otherwise the tripwire blocks mini+local-name). See `docs/superpowers/specs/2026-06-01-mac-mini-stack-migration-design.md`.

## Tech stack

- Python 3.13 via `uv` only (no bare `python`/`pip`/activated venvs)
- FastAPI + Pydantic v2, psycopg 3, APScheduler 3
- Next.js 16 + React 19, TypeScript, hand-rolled SVG charts (no chart library)
- Vitest + Playwright (web), pytest + pytest-postgresql (Python)
- Types flow API → client via `openapi-typescript` → `web/lib/types.ts`

## Daily commands

```bash
uv sync --extra postgres          # install
bash scripts/migrate.sh           # apply SQL migrations (idempotent)
bash scripts/dev.sh               # run web, API, 2 UW workers, and 2 massive workers
uv run pytest                     # python tests
cd web && npm run test            # vitest
cd web && npm run gen:types       # regenerate types.ts after API change
```

## Release procedure

Tag-driven, launchd-native (no Docker). Cut a release with
`scripts/release/cut.sh prepare [patch|minor|major]` (opens a release PR) → merge
→ `scripts/release/cut.sh tag` (pushes `vX.Y.Z`). The tag fires
`.github/workflows/release.yml` (verify → publish GitHub Release). The mini's
`com.argon.deploy-poller` (every 120s) deploys the latest **published,
non-prerelease** Release via `scripts/deploy/macmini-prod.sh`. Prereleases
(`vX.Y.Z-rc1`) verify + publish but never auto-deploy. See
`docs/runbooks/release.md`.

## Live spot WS feed (xenon primary / massive fallback)

The spot WS consumer (`uw_scan.worker.massive_ws_consumer` — module name retained for plist/dev.sh compat) connects to xenon's IB realtime server as the primary live feed and falls back to massive's WS automatically. Xenon streams 24h whenever IB Gateway is connected (massive only delivers Mon–Fri 04:00–20:00 ET). Failover triggers: connect failure, `ib_connected: false` at connect, or in-session tick silence (watchdog armed only inside massive's feed window — failing over outside it buys nothing). While on massive, a probe re-tries xenon every retry interval and switches back on recovery. `watchlist_card.spot_source` / `intraday_quote.source` tag each row (`xenon_ws` | `massive.com_ws`); `/api/health` `ws_consumer.active_source` shows the live feed.

- `XENON_WS_ENABLED` — primary-feed switch; default **false**
- `XENON_WS_URL` — default `ws://127.0.0.1:8765` (right for the mini, where xenon runs). MacBook dev points over Tailscale: `ws://100.66.147.98:8765`
- `XENON_WS_PORT_FILE` — default `/tmp/xenon-ib-realtime.json`; xenon writes its actual port there if 8765 is taken. Only consulted when the URL host is localhost; empty string disables
- `XENON_WS_RETRY_PRIMARY_SECONDS` — stay on massive this long after a xenon failure before re-probing; default 300
- `XENON_WS_QUIET_FAILOVER_SECONDS` — in-session silence threshold before failover; default 120; 0 disables
- `REGIME_WS_SYMBOLS` — always-subscribed regime symbols beyond the watchlist; default `VIX,VVIX,VIX3M,COR1M,SPX,HYG`. Feeds the live CRI/VCG compute (`/api/regime/{cri,vcg}/live`) and the 5-min `regime_live_scan` job (basis='live' rows in cri/vcg_snapshots; hourly :20/:25 scans stay the canonical basis='eod' dailies). Quotes older than `REGIME_LIVE_QUOTE_MAX_AGE_SECONDS` (default 900) are ignored — live endpoints then fall back to the EOD snapshot. `REGIME_LIVE_SCAN_INTERVAL_MINUTES` (default 5) sets the snapshot cadence. Nightly 03:40 ET `regime_live_validation` diffs the live-captured close vs the lake close (>0.5% → WARN). Massive fallback is stocks-only: indices stall during failover, HYG keeps ticking.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moremeds/argon](https://github.com/moremeds/argon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
