---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Go bot for market-making on Hyperliquid perpetual markets — both HIP-3 dex assets (e.g. `xyz:CL`) and standard perps (e.g. `BTC`, `ETH`). Always quotes both sides, uses short-window VWAP as the primary fair-value anchor, and earns bid-ask spread with mean-reversion on VWAP wicks.

**The shipped `config.example.yaml` targets a single asset (`xyz:CL`).** The bot's code still supports multi-asset / spread-pair mode (mixed HIP-3 + standard perps in one process, optional z-score cross-leg spread engine), and every relevant subsystem is per-asset aware (`Config.ForAsset(coin)`, per-asset optimizer/risk/MM overrides). To run multiple assets, add them to `strategy.assets` and — if they are correlated — leave `spread_engine_enabled` at its default (auto-on with 2+ assets) or set it explicitly.

Runtime config lives in `config.yaml` (copied from `config.example.yaml`, never committed). High-level features: adaptive notional, z-score calibration, trade-flow imbalance, post-fill decay, regime detection, markout tracking, per-side toxicity scoring with idle decay, inventory half-life controller, entry-side size reduction, dynamic refresh intervals, volume-triggered burst ladders, a unified utility-driven optimizer (4D state, policy table, counterfactual rollback), and a rate-limit-aware API budget allocator.

### Canonical Docs

- `docs/ALGORITHM.md` — full algorithm spec; **update this whenever the core algorithm changes**
- `docs/CONFIG.md` — per-field configuration reference
- `docs/GETTING_STARTED.md` — step-by-step operator walkthrough
- `config.example.yaml` — source of truth for the config schema (every field is annotated inline)
- `README.md` — user-facing overview and quick-start
- `AGENTS.md` is a symlink to `CLAUDE.md` — editing one updates the other

## Rules

- Any changes to the core algorithm should be documented by updating `docs/ALGORITHM.md` so it stays current
- All configs must print on startup so they land in the logs — keep this in mind when adding or removing configs (see `main.go` config-dump section)
- If you change any log fields or messages, update `cmd/monitor/parser.go` accordingly (or verify the monitor still parses correctly) — the TUI is a pure log consumer and will silently drop unknown fields

## Commands

Build artifacts land in `bin/` (gitignored).

```bash
make build                # go build -o bin/bot ./cmd/bot
make run                  # build + run bin/bot with config.yaml
make run-with-logs        # build + run bin/bot, tee to logs/logs.txt
make monitor              # build monitor TUI into bin/monitor
make run-monitor          # build + run bin/monitor (auto-discovers latest log in logs/)
make test                 # go test ./... -count=1
make test-race            # go test ./... -count=1 -race
make test-v               # go test ./... -count=1 -v
make vet                  # go vet ./...
make format               # go fmt ./...
make lint                 # go vet + staticcheck (if installed)
make validate             # build + validate config.example.yaml (bin/bot -validate; credential checks skipped so this works offline)
make check                # vet + validate + test-race
make clean                # rm -rf bin/
go run ./cmd/bot/ -config config.yaml     # run directly without building

# Run a single test
go test ./internal/spread/ -run TestEngineLongSpread -v

# Diagnostic: dump raw clearinghouse state (perps, dex, spot)
go run ./cmd/diag/

# Diagnostic: dump recent trades
go run ./cmd/diag-trades/

# Diagnostic: stream L2Book depth
go run ./cmd/diag-book/

# Monitor: TUI dashboard for real-time log monitoring
go run ./cmd/monitor/                    # auto-discover latest log in logs/
go run ./cmd/monitor/ logs/logs-cl-v11.txt  # open specific log file
```

## Architecture

Single binary, event-driven pipeline. All components communicate via typed Go channels with non-blocking sends (buffered channels, size 100). A single `context.Context` governs graceful shutdown via SIGINT/SIGTERM.

```
gRPC L2Book ──> fan-out ──> Spread Engine ──> continuous z-score ──> Quote Manager ──> Executor
                  │                                                       ^               │
                  ├──> Quote Manager (mid prices, book depth)             │               v
                  ├──> Live Executor (book snapshots for slippage close)  │         PositionUpdate
                  └──> Paper Executor (fill sim)                    RiskAction            │
                                                                         │               │
Trade gRPC    ──> Short VWAP ──> Quote Manager                    Risk Manager <─────────┘
              ├──> Long VWAP ──> Quote Manager
              └──> Trade-Flow Imbalance ──> Quote Manager

UserFills WS ──> Live Executor (fill reconciliation)
                  └──> Fill Callback ──> FillDecay, FillRateTracker, MarkoutTracker,
                                         SideToxicityTracker
```

### API Routing

- **QuickNode** -- gRPC streams (L2Book), WebSocket streams (UserFills, Trades), Info queries

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zer0cache/hyperliquid-market-maker-bot](https://github.com/zer0cache/hyperliquid-market-maker-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
