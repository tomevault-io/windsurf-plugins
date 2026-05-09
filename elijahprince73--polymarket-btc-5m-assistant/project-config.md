---
trigger: always_on
description: Guidance for Claude Code working on this repository.
---

# CLAUDE.md

Guidance for Claude Code working on this repository.

## Project overview

**Polymarket BTC 5m Assistant** — a Rust bot that trades Polymarket's 5-minute BTC Up/Down
contracts. Started life as a Node.js bot; this is the clean Rust rewrite. Paper-mode runs
end-to-end; live-mode is scaffolded but EIP-712 order signing is not yet wired. All trade
data targets the same Supabase that the `polymarket-dashboard` repo uses for analytics.

Goal shape: ~2k lines of Rust, 1s tick loop under 20ms, single static binary.

## Layout

```
src/
  main.rs         # wire everything, tokio runtime
  lib.rs          # re-exports for integration tests
  config.rs       # env parsing (see .env.example)
  model.rs        # Trade, OpenPosition, MarketSnapshot, Side, Mode
  engine/
    entry.rs      # pure fn: evaluate_entry → EntryDecision (7 SkipReason)
    exit.rs       # pure fn: evaluate_exit → ExitDecision (4 ExitReason)
    sizing.rs     # pure fn: size_trade (clamps min/max, rounds down to 0.01)
    state.rs      # EngineState + CircuitBreaker (3 losses → backoff)
    tick.rs       # 1s loop: snapshot → evaluate → executor.open/close → Supabase
  exec/
    mod.rs        # async_trait Executor + Open/Close req/result
    paper.rs      # PaperExecutor (fee, slippage, JSON ledger for crash recovery)
  data/
    gamma.rs      # Gamma /events?series_id={id} — flattens events.markets
    clob_rest.rs  # CLOB /price + /book (fallback only when WS stale)
    clob_ws.rs    # wss://ws-subscriptions-clob.polymarket.com/ws/market
                  # maintains best_bid/best_ask per token from book snapshots
  market/
    scheduler.rs  # event-driven: tokio::sleep_until(end_date + 2s) then refetch
  store/
    supabase.rs   # PostgREST: upsert_trade, list_trades, insert_signal_ticks
  api/
    routes.rs     # axum: /health, /status, /trades, /positions, /trading/*, /mode
static/           # vanilla HTML/JS UI served via tower-http ServeDir
tests/            # integration (paper round-trip); unit tests live next to each module
```

## Commands

```bash
cargo test                      # 37 tests (35 unit + 2 integration)
cargo check                     # fast type check
cargo run --release             # start server on :3000
RUST_LOG=debug cargo run        # verbose logs
```

## Key design decisions

1. **Executor trait is single-exec.** The engine holds `Arc<RwLock<Arc<dyn Executor>>>`.
   `POST /mode` rebuilds the inner `Arc<dyn Executor>`. The engine never branches on mode.

2. **Event-driven rollover, not polling.** `market::scheduler` `sleep_until`s the current
   market's `end_date + 2s`, then fetches once. A 5-minute `tokio::select!` backstop
   guards against clock drift / skipped markets. ~12 Gamma requests per hour vs. ~120 for
   polling.

3. **Pure fns in `engine/{entry,exit,sizing}`.** No I/O, no mutation. This is where all
   strategy logic lives; unit tests cover every `SkipReason` and `ExitReason`.

4. **Gamma query path:** `/events?series_id=10684&active=true&closed=false` → flatten
   `events[].markets` → `parse_gamma_market`. The `/markets?seriesSlug=` filter on Gamma
   is unreliable (observed returning unrelated markets), so we use the series_id path
   that the dashboard uses.

5. **Supabase is optional at boot.** If `SUPABASE_URL`/`SERVICE_ROLE_KEY` are missing,
   `SupabaseClient::new` returns a disabled client whose write methods are no-ops. The
   `/trades` route falls back to `state.recent_trades` (in-memory, capped at 100).

6. **`.env` variable names match the original JS bot.** `PRIVATE_KEY`, `CLOB_API_KEY`,
   `CLOB_SECRET`, `CLOB_PASSPHRASE`, `FUNDER_ADDRESS`, `CHAIN_ID`, `SIGNATURE_TYPE` — so
   the existing `.env` doesn't need to be rewritten.

7. **PORT precedence** (for DO App Platform): `PORT` → `HTTP_PORT` → 3000. DO injects
   `PORT` to match whatever `http_port` is declared in `.do/app.yaml`.

8. **Daily PnL is PST-scoped and Supabase-backed.** `EngineState.daily_pnl_date`
   tracks the current PST day; `maybe_roll_daily_pnl` resets when it flips. On boot,
   `boot_reconcile` (in `main.rs`) queries Supabase for `sum(pnl) WHERE exitTime >=
   midnight_pst_today` so the counter survives redeploys.

9. **Trades polling**: the UI calls `/trades` only on initial load and when a
   position transitions from set → null (i.e., right after a close). A manual
   *Refresh* button in the dashboard handles the edge cases. The status card still
   polls `/status` every 1.5s for the open-position mark and countdown.

10. **Graceful shutdown on SIGTERM**: `main.rs` listens on both `ctrl_c` and
    `SignalKind::terminate`. On shutdown: disable trading, best-effort close the open
    position (20s timeout), patch the Supabase row to `CLOSED / exitReason=shutdown`.
    DO App Platform triggers this on every redeploy.

11. **Boot-time reconciliation**: also in `main.rs::boot_reconcile`. If Supabase has
    an `OPEN` trade for this mode that survived a crash/SIGKILL, we patch it to
    `CLOSED / exitReason=abandoned_by_restart`. Live-mode needs a real CLOB
    `/positions` query here — not yet implemented.

## Conventions

- `rust_decimal::Decimal` for all money / price / share quantities. Never `f64`.
- `chrono::DateTime<Utc>` for timestamps; `chrono_tz::America::Los_Angeles` only inside

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ElijahPrince73/polymarket-btc-5m-assistant](https://github.com/ElijahPrince73/polymarket-btc-5m-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
