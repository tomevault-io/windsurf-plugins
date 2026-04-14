---
trigger: always_on
description: This file is the primary operating guide for future agent work in this repository. It describes what Exchangely is, how it works, and how to work in this codebase.
---

# Exchangely Agent Guide

This file is the primary operating guide for future agent work in this repository. It describes what Exchangely is, how it works, and how to work in this codebase.

## Document Roles

- `AGENTS.md` (this file): Project bible. What Exchangely is, architecture, current state, working rules, validation commands. Start here.
- `ROADMAP.md`: Forward-looking phased roadmap with checkboxes. What we're building next.
- `README.md`: Human-facing overview and quick-start guide.

If these documents disagree:

1. Current code and tests define actual behavior.
2. `AGENTS.md` defines how agents should work in this repo and describes the current system.
3. `ROADMAP.md` defines what to build next.
4. `README.md` is overview material, not the detailed implementation contract.

---

## What Is Exchangely

Exchangely is an event-driven crypto market data platform focused on historical OHLCV coverage for curated crypto/fiat and crypto/stablecoin pairs. It started as a "poor man's CoinGecko" for historical data availability and is evolving into a full-featured exchange rate platform with user accounts, portfolio tracking, price alerts, and a public API.

## Core Stack

- **Backend**: Go single-binary (API + planner + worker roles via `BACKEND_ROLE`)
- **Frontend**: React + Vite dashboard with SSE-driven realtime updates
- **Persistence**: TimescaleDB/PostgreSQL (authoritative source of truth)
- **Transport**: Kafka (task distribution + market events, not coordination authority)
- **Infrastructure**: Docker Compose local topology, GitHub Actions CI

## Architecture Principles

These are non-negotiable unless the user explicitly changes the architecture.

- PostgreSQL/Timescale owns system state. Do not move scheduling truth, sync state, or coordination responsibility into Kafka.
- Planner leadership is lease-based and DB-backed. Keep leadership and scheduling resilient to multiple instances.
- Workers must never mutate the same trading pair concurrently. Pair-level locking is a hard requirement.
- Historical backfill walks backwards from yesterday toward the past. No fixed start date; the system keeps walking back until providers return no data. `BACKEND_PLANNER_BACKFILL_BATCH_PERCENT` caps backfill tasks per planner cycle.
- A daily backfill probe task extends each pair one hour further into the past, discovering new provider data even after regular backfill exhausts known sources.
- `1h` is the preferred canonical historical resolution. Historical fetch granularity must never be coarser than `1d`.
- Month/year rollups, if added, must be derived from canonical stored candles, not provider-native month archives.
- Realtime tasks use a stable ID per pair (`live_ticker:{PAIR}:realtime`). Enqueue logic guarantees at most one pending/running realtime task per pair. Do not add poll-window timestamps to realtime task IDs.
- Prefer SSE for live UI updates when a stream already exists. Do not replace stream-driven flows with aggressive polling.
- The backend is intentionally one binary with role gating. Preserve that unless a deliberate architecture change is requested.
- Keep the project educational and local-first. Do not quietly harden it toward production assumptions without discussing the tradeoff.
- Auth is opt-in via `BACKEND_AUTH_MODE`. When empty, the auth middleware is bypassed and all endpoints are publicly accessible (matching pre-auth behavior). Valid modes are `local` (email/password only), `sso` (Google OAuth only), and `local,sso` (both). The backend validates that required variables are present for the chosen mode at startup and refuses to start if any are missing. When `BACKEND_AUTH_MODE` includes `sso` but `BACKEND_GOOGLE_CLIENT_ID` is empty, startup fails. When it includes `local` but `BACKEND_ADMIN_EMAIL` is empty, startup fails. `BACKEND_JWT_SECRET` is always required when any auth mode is set.
- When running behind a reverse proxy (nginx, Caddy, ALB, etc.), set `BACKEND_TRUSTED_PROXIES` to a comma-separated list of proxy CIDR ranges or IPs so that `X-Forwarded-For` / `X-Real-IP` headers are trusted for resolving the real client IP. This affects rate limiting, audit logging, and fail2ban-style IP blocking.

---

## Current Data Providers

| Provider            | Capabilities          | Notes                                      |
|---------------------|-----------------------|--------------------------------------------|
| Binance             | Historical + Realtime | OHLC + native `/ticker/24hr`               |
| Binance Vision      | Historical            | Bulk CSV archives                          |
| Kraken              | Historical + Realtime | OHLC + native `/Ticker`                    |
| CoinGecko           | Realtime              | `/simple/price` for live quotes            |
| CryptoDataDownload  | Historical            | Hourly/daily CSV fallback                  |

Default quote assets: `EUR` and `USD`.


## What's Built (Current State)

### Backend
- Single Go binary with planner/worker/API role gating
- DB-backed task & lease lifecycle with structured `slog` logging
- Backwards backfill strategy (yesterday → past, no fixed start date)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/block-o) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
