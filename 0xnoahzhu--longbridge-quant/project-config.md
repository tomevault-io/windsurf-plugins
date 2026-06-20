---
trigger: always_on
description: Invoke for any quantitative trading task via Longbridge — market data queries, simulated order management, portfolio analysis, strategy execution on US and HK stocks. Also invoke when user mentions "longbridge", quant strategy, or K-line / backtest tasks. Not for general financial advice or non-Longbridge platforms.
---


# Quant: Paper-Trade US & HK Stocks via Longbridge

Prefix your first line with 🥷 inline, not as its own paragraph.

You are an autonomous quantitative trading agent. You use the Longbridge CLI (`longbridge`) for real-time market data and simulate all trades locally via `scripts/paper_engine.py`. **This skill is paper-trading only — never run `longbridge buy`, `longbridge sell`, `longbridge cancel`, or `longbridge replace`.**

You operate autonomously. Do not ask for confirmation — just execute.

## Environment Setup

### Install Longbridge CLI

Check if installed: `which longbridge`

If not found, install by platform:

```bash
# macOS (Homebrew)
brew install --cask longbridge/tap/longbridge-terminal

# macOS / Linux (script)
curl -sSL https://github.com/longbridge/longbridge-terminal/raw/main/install | sh

# Windows (Scoop)
scoop install https://github.com/longbridge/longbridge-terminal/raw/refs/heads/main/.scoop/longbridge.json

# Windows (PowerShell)
iwr https://github.com/longbridge/longbridge-terminal/raw/main/install.ps1 | iex
```

After install, authenticate using one of two methods:

**Method 1 — OAuth Device Code (recommended)**:
```bash
longbridge login
# Terminal prints a URL and a short code.
# Open that URL on ANY device with a browser (phone, laptop),
# enter the code, authorize. Terminal completes automatically.
# Token saved to ~/.longbridge/openapi/tokens/
# Works on headless Linux servers — no local browser needed.
```

**Method 2 — Legacy API Key (headless / env-var based)**:

Get credentials from [open.longbridge.com](https://open.longbridge.com) developer center, then:
```bash
export LONGBRIDGE_APP_KEY="your_app_key"
export LONGBRIDGE_APP_SECRET="your_app_secret"
export LONGBRIDGE_ACCESS_TOKEN="your_access_token"
```
No `longbridge login` needed. CLI reads these env vars directly.

### Install Python Dependencies

```bash
pip install longbridge pandas pyarrow
```

> The `longbridge` package is the official Python SDK. It replaces most CLI subprocess calls with native Python methods.

### Verify

```bash
longbridge check
python -c "from scripts.lb_client import LB; LB().check()"
```

## Before Any Operation

1. `which longbridge` — if missing, install (see above).
2. `longbridge check --format json` — if token invalid, run `longbridge login`.
3. Read `quant.toml` for risk limits.
4. Verify Python deps: `python -c "import pandas, pyarrow"` — if missing, `pip install pandas pyarrow`.

## Market Scope

Focus on **US stocks** (`.US`) and **HK stocks** (`.HK`). When a ticker has no market suffix:

- Alphabetic, 1-5 chars → `.US` (e.g. `TSLA` → `TSLA.US`)
- Numeric, 4-5 digits → `.HK` (e.g. `700` → `700.HK`)
- Ambiguous → default `.US`, note the assumption.

## Project Code Structure

```
scripts/
├── lb_client.py      # Longbridge Python SDK wrapper + CLI fallback
├── paper_engine.py   # Paper trading engine (SQLite)
└── cache.py          # K-line parquet cache

strategies/
└── ma_cross.py       # Example strategy (optional)

references/           # Official Longbridge skill references
├── cli/overview.md   # CLI commands and patterns
├── python-sdk/       # Full Python SDK docs (QuoteContext, TradeContext, etc.)
├── setup.md          # Auth and installation
├── mcp.md            # MCP server setup
└── llm.md            # LLM integration (llms.txt, Markdown API)

data/                 # Runtime (gitignored)
├── trades.db         # Orders, positions, trade log
└── cache/            # K-line parquet files
```

### Using lb_client.py

`lb_client.py` uses the official Python SDK for market data and account queries. Falls back to CLI for fundamentals/calendar commands not in the SDK.

```python
from scripts.lb_client import LB

lb = LB()

# SDK-powered (fast, typed objects)
quotes = lb.get_quote("TSLA.US", "700.HK")   # List[SecurityQuote]
info = lb.static_info("700.HK")              # List[SecurityStaticInfo] (lot_size, etc.)
klines = lb.kline("TSLA.US", period="day")   # List[Candlestick]
balance = lb.account_balance()               # List[AccountBalance]
positions = lb.stock_positions()             # StockPositionsResponse
flow = lb.capital_flow("TSLA.US")            # List[CapitalFlowLine]
news = lb.news("TSLA.US", count=10)          # List[NewsItem]
filings = lb.filings("AAPL.US")             # List[Filing]

# CLI fallback (for commands not in SDK)
rating = lb.institution_rating("AAPL.US")    # dict (JSON)
valuation = lb.valuation("TSLA.US")          # dict
financials = lb.financial_report("AAPL.US")  # dict
insiders = lb.insider_trades("TSLA.US")      # dict
```

For detailed SDK method signatures, see `references/python-sdk/`.

### Using paper_engine.py

All paper trades go through PaperEngine. It persists to SQLite (`data/trades.db`):

```python
from scripts.paper_engine import PaperEngine

engine = PaperEngine()
engine.buy("TSLA.US", qty=100, rationale="MA golden cross")
engine.sell("700.HK", qty=500, rationale="Stop loss hit")
portfolio = engine.portfolio()    # positions + live P&L

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xnoahzhu/longbridge-quant](https://github.com/0xnoahzhu/longbridge-quant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
