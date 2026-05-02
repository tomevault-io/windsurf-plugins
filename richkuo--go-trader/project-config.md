---
trigger: always_on
description: - Requires Go 1.26.2 — install via Homebrew (macOS): `brew install go@1.26` or Linux tarball: `curl -sL https://go.dev/dl/go1.26.2.linux-amd64.tar.gz | tar -C /usr/local -xzf -`
---

# go-trader Project Context

## Environment
- Requires Go 1.26.2 — install via Homebrew (macOS): `brew install go@1.26` or Linux tarball: `curl -sL https://go.dev/dl/go1.26.2.linux-amd64.tar.gz | tar -C /usr/local -xzf -`
- Go is not in PATH via shell; use `/opt/homebrew/bin/go` directly (e.g. `cd scheduler && /opt/homebrew/bin/go build .`)
- Python venv at `.venv/bin/python3` (used by executor.go at runtime)
- In git worktrees, `.venv` is NOT copied — use the main repo's venv: `<main-repo>/.venv/bin/python3`
- Python deps managed with `uv` (see `pyproject.toml` / `uv.lock`)

## Quick Flow
- **New server:** tell OpenClaw `install https://github.com/richkuo/go-trader and init`.

## Setup
- `uv sync` — install Python deps into `.venv`
- Copy `scheduler/config.example.json` → `scheduler/config.json` and fill in API keys

## Repo Structure
- `scheduler/` — Go scheduler (single `package main`); all .go files compile together
  - `executor.go` — Python subprocess runner; max 4 concurrent, 30s timeout per script
  - `server.go` — HTTP status server (`/status`, `/health` endpoints)
  - `discord.go` — `discordgo.Session` wrapper for two-way Discord communication; `SendMessage`, `SendDM`, `AskDM` (blocking DM with timeout); `FormatCategorySummary` per-asset Discord messages; `fmtComma` — always pass absolute values
  - `init.go` — `go-trader init` interactive wizard + `--json <blob>` non-interactive mode; `generateConfig(InitOptions) *Config` is pure/testable; `runInitFromJSON(jsonStr, outputPath)` for scripted config gen (e.g. from OpenClaw); `runInit` orchestrates I/O
  - `prompt.go` — `Prompter` struct (String/YesNo/Choice/MultiSelect/Float/FloatRange); `FloatRange(prompt, default, min, max)` re-prompts on out-of-range input; inject `NewPrompterFromReader(r,w)` for tests
  - `updater.go` — update checker; `checkForUpdates(cfg, discord, &lastNotifiedHash, &mu, state)` — git fetch, channel notify + DM upgrade prompt (goroutine); `applyUpgrade(discord, ownerID, mu, state, cfg)` — git pull + go build + state save + restart; `restartSelf()` — systemctl → syscall.Exec fallback; logs `[update]` prefix
  - `correlation.go` — per-asset directional exposure tracking; `ComputeCorrelation` warns on concentration/same-direction thresholds
  - `config_migration.go` — `CurrentConfigVersion = 8`; auto-migrates config via Discord DM on startup; v8 migration strips dead `discord.spot_summary_freq` / `discord.options_summary_freq` fields and notifies owner
  - `balance.go` — balance tracking and capital management
  - `hyperliquid_balance.go` — Hyperliquid-specific balance sync (`syncHyperliquidAccountPositions`)
  - `leaderboard.go` — pre-computed strategy leaderboard for Discord summaries
  - `logger.go` — structured logging utilities
  - `notifier.go` — `MultiNotifier` wraps Discord + Telegram backends
  - `options.go` — options position management and expiry tracking
  - `portfolio.go` — portfolio-level aggregation and reporting
  - `risk.go` — per-strategy risk checks (drawdown limits, position sizing)
  - `telegram.go` — Telegram notification backend
  - `pricer.go` — `OptionPricer` interface; `ibkr_pricer.go` — IBKRPricer with Black-Scholes
  - `db.go` — SQLite state persistence (`modernc.org/sqlite` pure-Go driver); `OpenStateDB(path)`, `SaveStateWithDB`, `LoadStateWithDB`; tables: `app_state`, `strategies`, `positions`, `closed_positions`, `option_positions`, `closed_option_positions`, `trades`, `portfolio_risk`, `kill_switch_events`, `correlation_snapshot`; `InsertTrade` writes trades immediately via `tradeRecorder` hook (wired to `StateDB.InsertTrade` at startup) — trades survive mid-cycle crashes; `QueryClosedPositions(strategyID, symbol, since, until, limit, offset)` queries closed position history; `ClosedPosition` struct + transient `StrategyState.ClosedPositions` buffer flushed by `SaveState` inside the same transaction
  - `hyperliquid_marks.go` — `fetchHyperliquidMids(coins)` native Go HL `/info allMids` fetcher; correct oracle for HL perps PnL (replaces BinanceUS spot basis spoofing, issue #263)
  - `okx_marks.go` — `fetchOKXPerpsMids(coins)` native Go OKX `/api/v5/market/tickers?instType=SWAP` fetcher; USDT-margined swaps only (PR #280, issue #279); test stub via `okxMainnetURL` var
  - `deribit.go` — native Go Deribit `/public/ticker` fetcher (mark/underlying price + greeks)
  - `shared_wallet.go` — `SharedWalletKey{Platform, Account}` + `walletKeyFor(sc)`; prevents double-counting capital when multiple strategies trade from the same on-exchange wallet (currently HL live perps all share `HYPERLIQUID_ACCOUNT_ADDRESS`)
- `shared_scripts/` — Python entry-point scripts called by the scheduler
  - `check_strategy.py` — spot strategy signal checker
  - `check_options.py` — unified options checker (`--platform=deribit|ibkr|robinhood|okx`)
  - `check_price.py` — price check script
  - `check_hyperliquid.py` — Hyperliquid perps checker (`<strategy> <symbol> <timeframe> [--mode=paper|live]`; `--execute` for live orders)
  - `check_topstep.py` — TopStep futures checker (`<strategy> <symbol> <timeframe> [--mode=paper|live]`; `--execute` for live orders)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [richkuo/go-trader](https://github.com/richkuo/go-trader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
