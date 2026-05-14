---
trigger: always_on
description: Automated Polymarket prediction market trading bot built in Rust. **100% weather arbitrage** â€" uses NOAA + Open-Meteo forecasts + ensemble probabilities to find mispriced temperature markets and places limit orders at fair value.
---

# CLAUDE.md - Polymarket Weather Bot

## Project Overview
Automated Polymarket prediction market trading bot built in Rust. **100% weather arbitrage** â€" uses NOAA + Open-Meteo forecasts + ensemble probabilities to find mispriced temperature markets and places limit orders at fair value.

## Current Status (Mar 9, 2026)
- **Portfolio:** ~$94 | Cash: ~$72 | All-time P/L: **+$0.18** (breakeven)
- **Weather P&L:** **+$8.59** (9 markets won, 34 lost
- **Non-weather P&L:** -$8.41 (manual trades, US/Iran biggest drag)
- **PM2:** `polymarket-bot` ONLINE — Phase A deployed, schedule-aware scanning
- **PM2:** `polymarket-redeem` ONLINE — auto-redeem every 30 min via Builder relayer (gas-free)
- **Telegram:** Trade alerts + weekly P&L summary (Sundays midnight UTC)
- **polymarket-arb:** STOPPED (sniper/arb strategies paused)
- **Max Exposure:** $80 (raised from $60 after batch sell recovered funds)
- **Laddering:** DISABLED (34 orders placed, 0 fills)
- **Wellington:** REMOVED — worst city (-$14.02 on $17.32, 0 wins)
- **v9:** DEPLOYED Mar 9 — fill tracking reconciliation with Polymarket activity API (PR #15)
- **v8:** DEPLOYED Mar 7 — 8 bug fixes from full weather module code review (PR #12)
- **v7:** DEPLOYED Mar 5 — per-bucket hard cap ($4), SH seasonal bias correction, auto-exit position monitor
- **v7.1:** DEPLOYED Mar 5 — position monitor uses actual hours-to-resolution instead of position age
- **Phase A:** DEPLOYED Feb 28 — config tuning, bucket-type sizing, order book pricing
- **Phase B (backlog):** BUY_NO support + cross-bucket mispricing — after 3 days of Phase A data (Mar 3)
- **Auto-Redeem:** LIVE Mar 1 — Phase 2 Builder relayer (gas-free), replaces manual UI claiming
- **Station Codes Verified:** Chicago=KORD (O'Hare) ✅, Dallas=KDAL (Love Field) ✅ — confirmed against Polymarket resolution sources


### Mar 9 — v9: Fill Tracking Reconciliation (PR #15)

**Problem:** strategy_trades.json showed 0 fills and 0 wins across 62 trades. Actual Polymarket account data (from data-api) showed 39 fills and +$8.59 weather P&L.

**Root Cause (3 bugs):**
1. `check_fill_status()` skipped `resolved` trades, but trades were marked resolved before fills were checked
2. CLOB API is ephemeral — orders disappear after settlement, so fill checks returned "unfilled" for settled trades
3. No reconciliation with ground-truth account activity data

**Fix 1: Remove resolved skip in check_fill_status()**
- Changed from `if trade.dry_run || trade.resolved { continue; }` to `if trade.dry_run { continue; }`
- Trades now get fill-checked regardless of resolved status

**Fix 2: Add reconcile_with_api() function**
- New async function queries `data-api.polymarket.com/activity?user={proxy_wallet}` (paginates up to 500 activities)
- Filters for temperature-related TRADE/REDEEM activities
- Groups by eventSlug to calculate per-event P&L (sell + redeem - buy)
- Matches local trades by condition_id/token_id with cost similarity check (within 50%)
- Updates fill_status, fill_confirmed, pnl (proportional to event P&L), and outcome (WIN/LOSS)
- Marks stale unmatched trades (>48h) as NO_FILL
- Runs once per scan cycle after check_fill_status()

**Historical backfill:** Separate PowerShell script reconciled all 62 existing trades against 249 API activities.
Before: 0 fills, 0 P&L, 0 outcomes. After: 39 fills, 39 P&L, 46 outcomes (12 WIN, 27 LOSS, 4 NO_FILL, 3 MANUAL_CLOSE).

**Account analysis (from Polymarket API):**
- 249 total activities (206 trades + 43 redemptions) across 55 markets
- Weather: +$8.59 P&L (165 trades). Non-weather: -$8.41 P&L (41 trades)
- Probability model insight: winning markets avg prob 0.462 vs losing avg prob 0.410 — model doesn't separate winners from losers
- Worst cities by P&L: Buenos Aires (-$37), Dallas (-$23), Seoul (-$11) — candidates for removal
- Best cities: NYC (+$8.71), Ankara (+$2.19), Chicago (+$6.43)
### Mar 7 — v8: Bug Fixes from Full Code Review (PR #12)

**8 correctness fixes, no new features:**

**Task 1: 5-share guard replaced with 1-share sanity guard**
- Both the $1.00 dollar floor (v5) AND the old `shares < 5.0` guard were running simultaneously
- Toronto case: $1.34 cost at $0.30/share = 4.46 shares → passed dollar floor, blocked by share guard
- Fixed in both main trade pass and laddering pass in `run_once()`

**Task 2: Ensemble bias correction**
- `parse_multi_model` applied `bias_f`/`bias_c`; `fetch_ensemble` did not
- With current config (bias=0.0) no live impact, but a systematic offset risk for future config changes
- Fixed in `open_meteo.rs` `fetch_ensemble()`

**Task 3: Position monitor uses local city timezone**
- Resolution window was hardcoded to `23:59 UTC`; weather markets resolve at end-of-local-day
- Seoul (UTC+9) local midnight = 14:59 UTC — exit window was opening 9h late
- Added `local_end_of_day_utc(date, city_name)` helper using `chrono-tz` crate
- Added `chrono-tz = "0.9"` to `Cargo.toml`

**Task 4: Buffer check uses `adjusted_forecast.high_temp`**
- Same-day observation adjustments modified `adjusted_forecast` but buffer check still read raw `forecast.high_temp`
- One-line fix in `run_once()`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linuzri/polymarket-bot](https://github.com/linuzri/polymarket-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
