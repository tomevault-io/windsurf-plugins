---
trigger: always_on
description: This file provides guidance to Codex and other repo-aware coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex and other repo-aware coding agents when working with code in this repository.

## Project Overview

Optix is a **US stock & options strategy analysis tool** that helps identify sell-side opportunities for upcoming expirations. It combines real-time market data from Interactive Brokers (IBKR) with quantitative analysis powered by a Python gRPC engine.

**Architecture**: Hybrid Go + Python system
- **Go backend**: CLI tools, web server, IBKR integration, SQLite caching, gRPC orchestration
- **Python engine**: Technical analysis, options pricing (Black-Scholes), Greeks calculations, strategy recommendations

**Data flow**: IBKR TWS/Gateway → Go broker client → SQLite cache → Web UI or CLI → Python analysis engine (via gRPC) → Results

## Essential Commands

### First-Time Setup
```bash
# 1. Install Python dependencies (REQUIRED before first run)
python3 -m venv python/.venv  # requires Python 3.11+
python/.venv/bin/pip install -e 'python/[dev]'

# 2. Build Go binaries
make build
```

### Running the Application

**Start the web UI** (recommended for most users):
```bash
# Start backend server (default: http://127.0.0.1:8080)
./bin/optix-server --web-addr 127.0.0.1:8080

# OR use make
make run-server
```

**Start Python analysis engine** (required for analysis features):
```bash
# Terminal 1: Python gRPC server
make py-server

# OR directly
python/.venv/bin/python -m optix_engine.grpc_server.server --addr=localhost:50052
```

**CLI usage examples**:
```bash
# Get stock quote
go run ./cmd/optix-cli quote AAPL

# View option chain
go run ./cmd/optix-cli chain AAPL --expiry 2024-03-15

# Run full analysis
go run ./cmd/optix-cli analyze TSLA

# Launch dashboard
go run ./cmd/optix-cli dashboard

# Manage watchlist
go run ./cmd/optix-cli watch add NVDA
go run ./cmd/optix-cli watch list
```

### Development

**Run tests**:
```bash
# Go unit tests
go test ./...

# Python unit tests
python/.venv/bin/python -m pytest python/tests/ -v

# Integration tests (starts Python server, runs Go tests, stops server)
make test-integration

# Single test
go test -v -run TestAnalysisClient ./internal/analysis/
```

**Regenerate protobuf code** (after editing `.proto` files):
```bash
make proto
# OR
./scripts/proto-gen.sh
```

**Clean build artifacts**:
```bash
make clean  # Removes bin/ and data/optix.db
```

### IBKR Configuration

- Default connection: `127.0.0.1:4001` (IB Gateway live)
- `--ib-port` accepts aliases: `gateway` (4001), `tws` (7496), or a numeric port
- Override host with `--ib-host`
- Paper trading ports: `7497` (TWS) or `4002` (Gateway)

## Code Architecture

### Go Structure (`internal/`)

**`broker/`**: Abstraction layer for market data sources
- `broker.go`: Interface definition (`Connect`, `GetQuote`, `GetHistoricalBars`, `GetOptionChain`)
- `ibkr/`: Interactive Brokers implementation using `github.com/scmhub/ibapi`

**`analysis/`**: gRPC client to Python engine
- `client.go`: Go wrapper for `AnalysisService` (PriceOption, GetMaxPain, AnalyzeStock, BatchQuickAnalysis)
- Integration tests require Python server running

**`marketdata/`**: Multi-asset market snapshot layer (indices/futures/yields/vol/FX)
- Business-ID `AssetRef` + `Source`/`Router` abstraction routes by `AssetClass` to pluggable sources (currently all via yfinance; zero IBKR dependency)
- `PulseService` with 60s in-memory TTL + SQLite `market_pulse_bars` two-tier cache
- `earnings.go`, `option_chain.go`, and `raw_bars.go`: yfinance subprocess helpers for Market Intel earnings dates, Put/Call ratios, and raw-ticker pre/post bars

**`intel/`**: Market Intel scheduling plane (pure functions, zero IBKR/LLM)
- `phase.go`: four-phase market clock `PhaseAt`/`NextTransition`/`ViewFor` (premarket/intraday/postclose/closed)
- `calendar.go`: built-in NYSE 2026-2027 holiday/early-close calendar
- `handlers.go`: `GET /api/intel/state`, `/api/intel/pulse`, `/api/intel/journal`, `/api/intel/premarket/{overnight,gaps,movers,sentiment}`, `/api/intel/postclose/{earnings,timeline,read-across,movers}`, `/api/intel/event/{rates,diff,patterns,sensitivity}`, and `/api/intel/shock/{regime,fingerprint,analogs,liquidity}`

**`premarket/`**: Market Intel premarket analysis plane (pure compute, zero IBKR/gRPC)
- `overnight.go`: descriptive overnight transmission chain (N225→TSMC→SX5E→ES)
- `gaps.go`: SPX implied open + historical gap-fill distribution (migration 007 cache, lazy TTL)
- `movers.go`: watchlist ∪ curated liquid tickers, premarket move and volume-ratio ranking
- `sentiment.go`: Put/Call + VIX3M/VIX term premium, degraded regime label
- `service.go`: `PremarketService` bundle and per-card failure isolation for CLI/HTTP

**`postclose/`**: Market Intel postclose analysis plane (pure compute, zero IBKR/gRPC)
- `earnings.go`: yfinance EPS consensus/report rows for the earnings quick read
- `movers.go`: regular + after-hours + combined move extraction from raw 5m prepost bars
- `read_across.go`: same-sector read-across edges via the embedded sector map
- `timeline.go`: structured postclose event timeline
- `service.go`: `PostcloseService` bundle and per-card failure isolation for CLI/HTTP


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IS908/optix](https://github.com/IS908/optix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
