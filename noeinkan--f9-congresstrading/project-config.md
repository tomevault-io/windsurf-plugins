---
trigger: always_on
description: > **Migration complete:** the Streamlit dashboard has been replaced by a FastAPI JSON API (`src/api/`) + React frontend (`frontend/`). See **CLAUDE.md** for architecture and **PROJECT_INDEX.md** for the file map. This file remains authoritative for the Python data layer.
---

# Agent instructions — Congress Trading



> **Migration complete:** the Streamlit dashboard has been replaced by a FastAPI JSON API (`src/api/`) + React frontend (`frontend/`). See **CLAUDE.md** for architecture and **PROJECT_INDEX.md** for the file map. This file remains authoritative for the Python data layer.



## Project



Python tracker for U.S. House and Senate public financial disclosures: raw PDFs under `data/raw/`, normalized data in SQLite under `data/db/`, optional CSV exports, and a React dashboard backed by the FastAPI API. Full setup, legal notes, and CSV schema: **README.md**.



## Environment



- **Python**: 3.10+

- **Interpreter**: prefer project venv `.venv\Scripts\python.exe` (Windows) — see README troubleshooting if imports fail.

- **Secrets**: `POLYGON_API_KEY`; optional `OPENFIGI_API_KEY`. `re-resolve-tickers` requires at least one of these; set `CONGRESS_RE_RESOLVE_NO_KEY_OK=1` to bypass the check (e.g. CI runs with no network).

- **App auth (optional, VPS)**: `APP_USERNAME`, `APP_PASSWORD` (non-empty enables login gate), `APP_SESSION_SECRET`, `APP_SESSION_COOKIE`, `APP_SESSION_HTTPS_ONLY`, `APP_SESSION_MAX_AGE`, `APP_CORS_ORIGINS`.

- **API server**: `API_SERVER_ADDRESS` (default `127.0.0.1`), `API_SERVER_PORT` (default `9001`), `API_RELOAD=1` for dev autoreload.

- **House PTR autodownload** (defaults in `src/config.py`): `HOUSE_PTR_AUTO_DOWNLOAD`, `HOUSE_PTR_AUTO_DOWNLOAD_MIN_YEAR`, `HOUSE_PTR_AUTO_DOWNLOAD_MAX_YEAR`, `HOUSE_PTR_DOWNLOAD_MIN_INTERVAL_SECONDS`. Be conservative with Clerk traffic; do not bulk-hammer `disclosures-clerk.house.gov`. Senate eFD has its own terms — README.



## Entrypoint



All CLI: **`python -m src.main <command>`** from repo root.



| Command | Purpose |

|--------|---------|

| `download-house-fd` | Bulk House FD zips → `data/raw/house/` (`--years`, `--overwrite`, `--zip-only`) |

| `download-oge` | OGE Executive PDFs (278-T + 278e) → `data/raw/oge/<doc_id>.pdf` from the hard-coded registry in `src/oge_source.py` (`--filer NAME`, `--overwrite`). Conservative 1 req/sec; fails loud on 404. |

| `ingest-house` | House PTR + FD pipeline |

| `ingest-senate` | Senate PTR (expects PDFs present) |

| `ingest-oge` | OGE Executive (278-T + 278e) PDFs from `data/raw/oge/`. 278-T rows go to `transactions`; 278e rows go to the dedicated `executive_holdings` table. |

| `ingest-all` | House + Senate + OGE |

| `export-csv` | Normalized trades → `--out` (default `data/congress_trades.csv`). Optional Polygon columns: `--polygon-pnl` [`--as-of YYYY-MM-DD`] [`--polygon-pnl-cache-only`] [`--polygon-pnl-refresh`] |

| `warm-polygon-price-cache` | Prefetch Polygon daily bars into `polygon_daily_bar_cache` for tickers in `transactions` (`--as-of`, `--refresh`, `--cache-only`, `--progress-every N`) |

| `export-fd-csv` | FD report CSV |

| `export-review-csv` | Review queue CSV |

| `re-resolve-tickers` | Re-resolve ticker/issuer on all SQLite transactions without re-parsing PDFs; uses disclosure text + cache/API (`--clear-asset-cache`) |



**Run the dashboard:** `python -m src.api` (API) + `cd frontend && npm run dev` (Vite dev server). See `frontend/README.md` and `deploy/README.md` for production (Caddy + static build).



Windows bootstrap: `powershell -ExecutionPolicy Bypass -File .\bootstrap.ps1 <same-subcommand>`.



## Layout



- **`src/`** — application code (core modules below).

- **`src/api/`** — FastAPI JSON API (routers, repository, analytics). No Streamlit imports.

- **`frontend/`** — React dashboard (Vite + Mantine + TanStack Query/Table + ECharts). Dev: `npm run dev` proxies `/api/*` to the FastAPI service. See `frontend/README.md`.

- **`data/raw/house/`**, **`data/raw/senate/`** — PDFs (and zips; pipeline may extract).

- **`data/db/`** — SQLite.

- **`data/cache/`** — ticker resolution cache.

- **`tests/`** — pytest test suite; run with `pytest` from repo root.



### Core modules in `src/`



| Module | Responsibility |

|--------|---------------|

| `config.py` | Paths, env vars, constants |

| `db.py` | SQLite connection, schema init, shared queries |

| `utils.py` | Shared utilities (`normalize_key`, text helpers) |

| `main.py` | CLI entrypoint (argparse) |

| `ingest_house.py` | House PTR + FD ingest pipeline |

| `ingest_senate.py` | Senate PTR ingest pipeline |

| `ingest_oge.py` | OGE Executive (278-T + 278e) ingest pipeline (`chamber='Executive'`) |

| `oge_source.py` | Hard-coded OGE filing registry (dataclass + `TRUMP_OGE_FILINGS`) |

| `download_oge.py` | Conservative OGE PDF downloader (1 req/sec) |

| `parse_ptr.py` | PTR PDF parsing |

| `parse_fd.py` | Financial Disclosure PDF parsing |

| `parse_oge.py` | OGE 278-T (periodic) + 278e (annual) PDF parsing |

| `download_house_fd.py` | Bulk download from House Clerk |

| `ticker_lookup.py` | Ticker/CUSIP resolution |

| `issuer_enrichment.py` | Issuer metadata enrichment |

| `polygon_prices.py` | Polygon.io daily bar fetching + cache |

| `export_csv.py` | CSV export logic |

| `house_coverage.py` | House coverage tracking |



### `src/api/` modules




<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Noeinkan/F9_CongressTrading](https://github.com/Noeinkan/F9_CongressTrading) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
