---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A monorepo of Go tools for Indian investors, all zero-dependency (Go stdlib only), Go 1.26,
in one `go.work` workspace:

- **`itr-foreign/`** — a CLI that turns Interactive Brokers (IBKR) holdings into Indian ITR
  schedules: `fa` builds **Schedule FA** (Foreign Assets, calendar year) and `fsi` builds
  **Schedule FSI + TR** (foreign income and tax relief, financial year) with a Form 67
  worksheet. The most developed tool; most of this doc is about it.
- **`correlation/`** — computes return **correlations** across assets to gauge portfolio
  diversification (CSV/Yahoo-driven, md/csv/json output).
- **`backtest/`** — an offline **backtester** for rule-based strategies on NSE daily data
  (trend, momentum, mean-reversion and breakout rules vs a buy-and-hold benchmark, realistic
  costs). Each strategy lives in its own file under `internal/strategy/` and is registered in
  `pipeline.buildStrategy`; `--strategy all` compares them in one table, `--sort` ranks it, and
  `--vol-target` adds a volatility-targeting position-sizing overlay (the benchmark stays
  unscaled). A `walkforward` command splits history into out-of-sample folds (with `--optimize`
  re-fitting parameters per fold), and a `sweep` command maps the metric surface over a 1-D/2-D
  parameter grid (plateau vs overfit spike), a `montecarlo` command bootstraps daily returns to
  gauge how much of a result is luck, and a `regime` command splits performance by market state
  (bull/bear, high/low vol). A research tool: output is not advice, and a backtest is a hypothesis
  fit to the past, not a forecast. See `backtest/README.md`.
- **`papertrade/`** — runs a strategy **forward** on live Yahoo data with **simulated** fills and
  a persistent account (`account.json`, `fills.jsonl`, `equity.jsonl` under a `--dir`). Commands:
  `init`, `step` (act on the newest unprocessed daily bar; idempotent per bar; records a daily
  marked-to-market equity snapshot even with no trade), `status` (marked to market), `summary`
  (performance over the tracked period vs a buy-and-hold benchmark, via `internal/perf`),
  `history`, `list` (multi-account overview under a `--root`), and `export` (equity/fills to CSV).
  Reuses copies of backtest's `strategy` and `yahoo` packages (kept in sync manually,
  per the isolated-module convention); a `broker.PaperBroker` mirrors the backtest cost model and
  buys are capped to available cash. Places **no real orders**. See `papertrade/README.md`.

## Commands

This is a `go.work` workspace. **Run all `go` commands from inside `itr-foreign/`** — `go ...
./...` from the repo root fails (`directory prefix . does not contain modules listed in
go.work`). `go` here is installed via **asdf** (`~/.asdf/shims/go`), so it's only on PATH in a
login shell; non-login shells may need the full shim path.

```sh
cd itr-foreign
go test ./...                          # all tests
go test -race ./...                    # what CI runs
go test ./internal/peak -run Compute   # a single package / test
go vet ./...
gofmt -l .                             # CI fails if this prints anything; gofmt -w . to fix
go build ./cmd/itrforeign
go run ./cmd/itrforeign fa --year 2026 --statement <file.xml> --rates <ttbr.csv> [--prices <p.csv>] [--entities <e.csv>]
go run ./cmd/itrforeign fetch-prices --year 2026 [--tickers <file>] [--out <file>]  # Yahoo daily closes → prices CSV
go run ./cmd/itrforeign fsi --fy 2025-26 --statement <file.xml> --rates <ttbr.csv> --tin <TIN> --marginal-rate 30
```

Golden tests (they lock the whole offline render path for **both** schedules — `report.*` for
FA and `report-fsi.*` + `schedule-fsi.json` for FSI): after an **intended** output change,
`go test ./internal/pipeline -update`, then review the diff of
`internal/pipeline/testdata/golden/*` before committing. Never blind-update.

CI: `.github/workflows/ci.yml` runs gofmt + vet + build + `test -race` on the module.

## Architecture

Pipeline (one stage per package, lower-level deps only):

```
                    ibkr (parse Flex XML / online pull)  →  model.Statement
                                        │
        ┌───────────────────────────────┴────────────────────────────────┐
        │ Schedule FA — CALENDAR year                 Schedule FSI/TR — FINANCIAL year
        │   fx (SBI TTBR at the event date)             rule115 (TTBR at the month end
        │   peak (per-security + true A2 NAV peak)        BEFORE the event; 128(8) for tax)
        │        ↓                                      gains (24-month term, 23-Jul-2024
        │   fa.Build (Tables A2 + A3)             split, per-leg vs net-gain FX)
        │                                                     ↓
        │                                              fsi.Build (country × head + TR + Form 67)
        └───────────────────────────────┬────────────────────────────────┘
                                 report (md/csv/json/html, + ITD JSON fragment)
```

`internal/itr` holds the lookups the ITR itself defines (country codes, IB entity metadata) and
is shared by both branches, so FA and FSI can never disagree about a country code.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akagr/finance-tools](https://github.com/akagr/finance-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
