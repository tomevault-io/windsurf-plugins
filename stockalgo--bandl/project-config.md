---
trigger: always_on
description: > **Audience:** AI coding agents. Read this file only; do not scan `lib/` unless something is missing here.
---

# bandl — agent reference

> **Audience:** AI coding agents. Read this file only; do not scan `lib/` unless something is missing here.
> **Humans:** use [README.md](README.md).

### How agents get this doc (not via PyPI)

`AGENTS.md` lives in the **Git repository only** — it is **not** installed by `pip install bandl`. Users point their agent at a stable GitHub URL; the agent reads the doc, then uses the installed package.

| What | Action |
|------|--------|
| **Library** | `pip install bandl` (PyPI) or `pip install git+https://github.com/stockalgo/bandl.git` |
| **Agent instructions** | Attach or paste one link below (pin a tag/branch for reproducibility) |

**Stable links (replace `master` with a tag, e.g. `v0.4.0`, when you need a fixed version):**

- Browse: `https://github.com/stockalgo/bandl/blob/master/AGENTS.md`
- Raw (fetch): `https://raw.githubusercontent.com/stockalgo/bandl/master/AGENTS.md`

**Example user prompt to an agent:**

> Use bandl for this task. Follow the API and recipes in:
> https://github.com/stockalgo/bandl/blob/master/AGENTS.md
> Install with `pip install bandl` if needed.

In Cursor / similar IDEs you can also `@AGENTS.md` from a cloned repo, or add the raw URL to project rules.

---

## When to use bandl

Use **bandl** for **historical OHLCV** and **live trading/portfolio** from one sync Python client with normalized models and pandas output:

- **Crypto spot/perp** (Binance, CoinDCX)
- **Indian equities & indices** (Zerodha)
- **Options** — NSE/BSE F&O and MCX commodities, **including expired contracts** (Dhan)
- **Broker account history** — orders, fills, ledger, PnL (CoinDCX, Zerodha)
- **Live trading & portfolio** — place/modify/cancel orders, live order state, positions, holdings, balances, margin (Zerodha, Dhan)

bandl is **sync HTTP only** — no WebSockets, no async client.

**Supported domains:**

| Domain | bandl surface | Providers |
|--------|---------------|-----------|
| Crypto spot/perp OHLCV | `client.crypto.*` | `binance` (default), `coindcx` |
| Indian equity/index OHLCV | `client.equity.*` | `zerodha` (auth) |
| Option OHLCV / chain / expiries | `client.derivatives.*` | `dhan` (auth) |
| Account orders/fills/ledger/PnL (history) | `client.account.*` | `coindcx`, `zerodha` (auth) |
| Live order write/read | `client.trade.*` | `zerodha`, `dhan` (auth) |
| Live positions/holdings/balances/margin | `client.portfolio.*` | `zerodha`, `dhan` (auth) |
| Symbol discovery | `client.list_symbols(source=...)` | per provider |

**Not in bandl:** live WebSockets, US equities, async client, crypto trading (binance/coindcx `client.trade`/`client.portfolio` not wired). CoinDCX futures candles: no M3/H2/H6. Binance = USDT-M perpetuals only. Dhan option OHLCV intervals: only 1, 5, 15, 60 min. Live trading covers **regular-variety orders only** — no AMO/CO/BO/iceberg/GTT/Forever/slicing/margin-preview/convert-position yet (see "Live trading — what's not yet wired").

---

## Install & bootstrap

```bash
pip install bandl
```

- Python **3.10+**
- Default install includes: `httpx`, `pydantic`, `pandas`, `requests`, …

```python
from datetime import datetime, timedelta, timezone

from bandl import Bandl, BandlConfig, Interval, ProviderSettings

client = Bandl()
end = datetime.now(timezone.utc)
start = end - timedelta(days=30)
```

If `start` / `end` omitted on OHLCV or account calls → defaults to **last 30 days** ending now (UTC).

---

## Client shape

```
Bandl(config?: BandlConfig)
├── .crypto          → _Facet           (default_source = config.default_crypto_provider, usually "binance")
├── .equity          → _Facet           (default_source = config.default_equity_provider, usually "zerodha")
├── .derivatives     → _DerivativesFacet (default_source = config.default_derivatives_provider, usually "dhan")
├── .account         → AccountFacet     (history: orders/fills/ledger/pnl — read-only, past data)
├── .trade           → TradeFacet       (live: place/modify/cancel + open orders/order/today's trades)
├── .portfolio       → PortfolioFacet   (live: positions/holdings/balances/margin)
├── .get_ohlcv(...)                  # low-level; prefer facets
├── .get_ohlcv_dataframe(...)
├── .get_option_ohlcv(...)           # low-level; prefer client.derivatives
├── .list_expiries(...)
├── .get_option_chain(...)
├── .list_symbols(source=..., search=..., limit=..., asset_type=...)
├── .get_24hr_tickers(source=..., asset_type=...)  # Binance / CoinDCX USDT-M futures
├── .list_providers()         # ["binance", "coindcx", "dhan", "zerodha"]
└── .configure_provider(name, ProviderSettings)
```

**Facet shortcuts** (same signatures; `source` defaults to facet provider):

- `client.crypto.get_ohlcv`, `get_ohlcv_dataframe`, `list_symbols`, `get_24hr_tickers`
- `client.equity.get_ohlcv`, `get_ohlcv_dataframe`, `list_symbols`
- `client.derivatives.get_ohlcv`, `get_ohlcv_dataframe`, `list_expiries`, `get_option_chain`

---

## Decision tree: pick provider

```
USER WANTS MARKET CANDLES?
├─ Crypto (spot)
│  ├─ Try source omitted or "binance" → client.crypto.get_ohlcv_dataframe(...)
│  └─ On GeoRestrictionError (HTTP 451) → source="coindcx"
├─ Crypto futures/perp OHLCV

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stockalgo/bandl](https://github.com/stockalgo/bandl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
