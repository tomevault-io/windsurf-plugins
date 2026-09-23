---
trigger: always_on
description: Free-tier data extraction from the **Finnhub** REST API. The shared API key lives
---

# Finnhub Data Project

Free-tier data extraction from the **Finnhub** REST API. The shared API key lives
in `.env` as `FINNHUB_API_KEY` and is loaded by `python-dotenv` in every script.

- **Base URL**: `https://finnhub.io/api/v1`
- **Auth**: every request carries `?token=<FINNHUB_API_KEY>` as a query param
- **Rate limit**: 60 req/min on free tier
- **Source of truth**: the live Swagger spec at
  `https://finnhub.io/static/swagger.json` (114 documented endpoints)
- **Re-probe**: `python probe_endpoints.py` hits every endpoint and bucketing it
  as `ok / empty / forbidden / other`. Run after any plan change.

## Project layout

```
finhub data/
├── .env                 # FINNHUB_API_KEY=...  (real key, gitignored — see .env.example)
├── finnhub_free.py      # exercises every endpoint the free key can reach
├── probe_endpoints.py   # auto-probe against the live Swagger spec
├── dashboard/           # Streamlit research dashboard (see below)
└── CLAUDE.md            # this file
```

## Dashboard

`dashboard/` is a local Streamlit app over the same 27 free endpoints. Type
any ticker in the sidebar and see a 6-tab research view. Uses `../.env`
for the API key and a SQLite cache at `dashboard/.cache/finnhub.db` to stay
under the 60 req/min free-tier rate limit.

```
cd "C:\Users\thesh\Desktop\finhub data\dashboard"
pip install -r requirements.txt   # adds streamlit
streamlit run app.py
```

| File | Purpose |
|---|---|
| `dashboard/app.py` | Streamlit UI: sidebar (symbol, refresh, asset class) + 6 tabs |
| `dashboard/finnhub_client.py` | Typed per-endpoint functions returning `Optional[...]` (never raise) |
| `dashboard/cache.py` | SQLite TTL cache, key = sha1(path + sorted params) |
| `dashboard/ai_client.py` | OpenRouter-backed AI analysis (button-triggered, cached) |
| `dashboard/.streamlit/config.toml` | Theme (dark) + server config |
| `dashboard/requirements.txt` | `streamlit>=1.30`, `requests>=2.28` |
| `dashboard/README.md` | Quick-start |

The 7 tabs:

1. **Overview** — quote card (price, change, day range, prev close), profile, peers, market status, daily-bar plotly chart with annotation that historical OHLCV is paid-tier only
2. **Financials** — `/stock/metric` table, `/stock/earnings` table, latest 10-K XBRL summary (Net sales, Net income, Total assets, Equity, etc.)
3. **Analyst & Insider** — `/stock/recommendation` stacked bar chart (6 months), top 25 `/stock/insider-transactions`, `/stock/insider-sentiment`
4. **Government / Alt-Data** — `/stock/lobbying`, `/stock/uspto-patent`, `/stock/visa-application`, `/stock/usa-spending`
5. **Filings & News** — `/stock/filings` (form, date, link) + `/news?category=general` (top 20 headlines, clickable)
6. **Reference** — IPO calendar, FDA calendar, market holidays, country list, COVID-19 US, symbol universe, plus crypto/forex reference when those asset classes are selected
7. **AI Analysis** — button-triggered OpenRouter analyses (overall brief, financials & earnings, government/alt-data, insider sentiment). Reads `OPENROUTER_API_KEY` + `OPENROUTER_MODEL` from `.env`; default `meta-llama/llama-3.3-70b-instruct:free` (free-tier — paid models 402 on a $0 account). Free `:free` models 429 unpredictably at peak, so `_chat()` falls back through `[nvidia/nemotron-3-super-120b-a12b:free, openai/gpt-oss-120b:free, qwen/qwen3-next-80b-a3b-instruct:free]` until one responds. Cached 6h, deterministic on (symbol, section, model). Cleared by the "Refresh this symbol" button.

**TTL policy** (per-endpoint, in seconds):

| Family | TTL |
|---|---|
| `/quote` | 60 |
| `/news`, `/stock/market-status` | 300 |
| All other per-symbol | 3600 |
| Reference data (country, COVID, FDA, IPO, exchanges, `/stock/symbol`) | 86400 |

**Use the client outside Streamlit:**

```python
from dashboard.finnhub_client import quote, financials_reported, latest_xbrl_summary
print(quote("AAPL"))                              # {'c': 293.08, ...}
fr = financials_reported("AAPL", freq="annual")    # long-format XBRL DataFrame
print(latest_xbrl_summary(fr))                    # latest 10-K headline figures
```

To wipe the cache, delete `dashboard/.cache/finnhub.db` or hit the
"Refresh this symbol" button in the sidebar.

## Shared helpers

```python
import os, requests
from dotenv import load_dotenv

load_dotenv()                      # picks up .env in the cwd
API_KEY = os.getenv("FINNHUB_API_KEY")
BASE    = "https://finnhub.io/api/v1"

def get(path, **params):
    params["token"] = API_KEY
    r = requests.get(f"{BASE}{path}", params=params, timeout=10)
    r.raise_for_status()
    return r.json()
```

## Free-tier endpoint catalog (27 of 114)

Every endpoint below returned 200 + non-empty data on 2026-06-24 against the key
in `.env`. The remaining 87 endpoints in the Swagger spec are paid-tier and
return HTTP 403 (`You don't have access to this resource`).

### Core stock data

| Method | Path | Required params | Returns |
|---|---|---|---|
| GET | `/quote` | `symbol` | `{c, d, dp, h, l, o, pc, t}` current/daily price |
| GET | `/search` | `q` | `{count, result:[{symbol, description, type, displaySymbol}]}` |
| GET | `/stock/profile2` | `symbol` | `{name, ticker, exchange, currency, country, finnhubIndustry, marketCapitalization, ...}` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [The-Options-Guru/finnhub-free-dashboard](https://github.com/The-Options-Guru/finnhub-free-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
