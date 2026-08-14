---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
# Install dependencies (use ci for deterministic installs)
npm ci

# Run in development mode (starts Vite dev server, then Tauri)
npm run tauri dev

# Build for production (cross-platform: .exe on Windows, .dmg on macOS, .deb/.AppImage on Linux)
# Requires signing env vars for updater artifacts:
#   $env:TAURI_SIGNING_PRIVATE_KEY = Get-Content "$env:USERPROFILE\.tauri\quant-desktop.key"
#   $env:TAURI_SIGNING_PRIVATE_KEY_PASSWORD = "your-password"
npm run tauri:build

# Alternative: cross-platform build with proxy auto-detect (see scripts/build.mjs)
node scripts/build.mjs

# Type-check the frontend
npx vue-tsc --noEmit

# Build Rust backend only
cargo build --manifest-path src-tauri/Cargo.toml
```

There are no dedicated test or lint commands configured yet. `vue-tsc` with the strict tsconfig enforces type correctness; `cargo build` compiles the Rust side.

## Architecture

QuantDesktop is a **Tauri 2 desktop app** for monitoring Chinese A-share stock markets. It has two separate webview windows driven by two Vite/Vue entry points, a SQLite-backed Rust backend, and a pluggable data source layer for fetching market data.

### Two-window layout

| Window | Label | Entry | Config |
|--------|-------|-------|--------|
| Main UI | `main` | `index.html` → `src/main.ts` | 1100×680, starts hidden, hides to tray instead of closing, position/size persisted to SQLite |
| Ticker bar | `ticker` | `ticker.html` → `src/ticker.ts` | 230×38, always-on-top, decorationless, positioned bottom-right, skip-taskbar |

Vite is configured with two Rollup inputs (`index.html` + `ticker.html`) in [vite.config.ts](vite.config.ts). Each entry mounts a separate Vue app with its own Pinia instance. Both share the same stores and composables via import.

### Rust backend (`src-tauri/src/`)

**`lib.rs`** — Application setup. Initializes SQLite database, registers data source adapters (Tencent first as default, then Sina as fallback), restores quote cache from DB, spawns the background polling `Scheduler` (with adaptive polling: probe → normal → idle for holiday detection), builds the system tray menu (left-click toggle, right-click menu with show/toggle-ticker/quit), registers all Tauri IPC commands, and sets up the auto-updater. The main window's `CloseRequested` event is intercepted to hide instead of quit. Window position/size is saved to SQLite and restored on next launch with monitor-boundary validation.

**`domain/mod.rs`** — Shared data types serialized across the IPC boundary to TypeScript types in [src/types/index.ts](src/types/index.ts):
- `Market` enum (CN/HK/US)
- `Quote` — real-time quote with price, change, open/high/low, volume, turnover, turnover_rate
- `IndexQuote` — index-level quote
- `Depth` — 5-level bid/ask depth (bids: `Vec<Level>`, asks: `Vec<Level>`)
- `Level` — single depth level (price + volume)
- `MinuteData` — intraday minute bar (time, price, open, high, low, volume, avg_price)
- `KLineData` — daily/weekly/monthly K-line bar (date, open, high, low, close, volume, turnover)
- `StockBrief` — minimal stock identifier for search results

**`db/mod.rs`** — SQLite database (via `rusqlite` with bundled SQLite). Three tables: `watchlist`, `settings` (key-value), `quote_cache`. Database is stored at `{app_data_dir}/quant-desktop.db`. Auto-creates tables and default settings on first open. `init_defaults()` inserts default settings only on first run (when key does not exist); user preferences persist across restarts.

**`datasource/mod.rs`** — Pluggable data source architecture. The `DataSource` trait defines `fetch_realtime()`, `fetch_indices()`, `search()`, `fetch_depth()`, `fetch_minute_data()`, `fetch_kline()`, `health_check()`. `DataSourceManager` holds a registry of adapters and an `active` name, supporting runtime switching. A `tokio::sync::Notify` wakeup mechanism triggers immediate refresh on data source switch.

Volume/turnover normalization: adapters return raw data in 手 (hands) / 万元 for stocks, and data is normalized to 股 (shares) / 元 via `normalize_volume(×100)` and `normalize_turnover(×10000)`. The frontend `formatVolume()` converts back to human-readable units for display.

- `sina.rs` — Sina Finance (新浪财经) adapter, the **backup** data source. GBK-encoded responses, `var hq_str_xxx="..."` format. Handles code-to-exchange mapping (sh/sz prefix). **Index fetching uses stock-format API** (codes without `s_` prefix, 30+ fields per line) because the compact index-only format (`s_` prefix, 6 fields) returns incorrect volume/turnover for 创业板指 (s_sz399006). **Shanghai (`sh`) index volume is consistently 1/100 of the correct value** in all Sina formats — corrected with `saturating_mul(100)` before entering the shared pipeline. Stock-format data arrives in 股/元 pre-normalized, so `normalize_volume`/`normalize_turnover` are NOT applied for indices. Search only supports exact 6-digit code lookup. Depth data fetched via Tencent API fallback (Sina's native depth endpoint is dead). Minute/K-line data from `money.finance.sina.com.cn`. Covers 7 major indices.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Leaderxin/quant-desktop](https://github.com/Leaderxin/quant-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
