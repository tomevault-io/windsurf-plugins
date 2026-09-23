---
trigger: always_on
description: CLI tool that automates regional pricing for App Store in-app purchases and subscriptions. Distributed on PyPI as `appstore-ppp-prices`; the module is `appstore_ppp_prices` and it installs two equivalent commands, `appstore-ppp-prices` and the shorter `ppp-pricing`. Calculates optimal prices for 175+ countries based on GDP per capita and optionally uses GPT to adjust coefficients per app type.
---

# CLAUDE.md — Project Context for AI Agents

## What This Project Does

CLI tool that automates regional pricing for App Store in-app purchases and subscriptions. Distributed on PyPI as `appstore-ppp-prices`; the module is `appstore_ppp_prices` and it installs two equivalent commands, `appstore-ppp-prices` and the shorter `ppp-pricing`. Calculates optimal prices for 175+ countries based on GDP per capita and optionally uses GPT to adjust coefficients per app type.

## Tech Stack

- Python 3.10+
- httpx — HTTP client for App Store Connect API
- PyJWT — JWT token generation for API auth
- openai — GPT integration for pricing analysis
- python-dotenv — .env config loading

## Architecture

```
appstore_ppp_prices/          — the installable package (importable as appstore_ppp_prices)
  cli.py          — Entry point, argument parsing, orchestration
  appstore.py     — App Store Connect API client (JWT auth, products, prices)
  pipeline.py     — Core workflow: find product → AI analysis → calculate → resolve → apply
  pricing.py      — Pure pricing logic: coefficients × US price → target prices
  countries.py    — Country data loader from CSV (GDP, categories, coefficients)
  ai_analyzer.py  — Prompt template, response parsing, caching, cache clearing
  llm.py          — OpenAI-compatible chat-completions client: retries, timeouts, provider override
  display.py      — Terminal output formatting, dry-run tables
  paths.py        — Config and cache directory resolution
  countries.csv   — 175+ countries with GDP per capita and default coefficients (shipped as package data)

.env              — Secrets (not committed)
.env.example      — Template for .env
```

**Never anchor runtime paths to `Path(__file__).parent.parent`** — in an installed
wheel that resolves to `site-packages`, not the repository. Data files go through
`importlib.resources`; writable state goes through `paths.py`.

## Key Patterns

- **Business logic is pure**: `pricing.py` has no I/O, no side effects
- **API client is stateful**: `AppStoreConnectClient` manages JWT token lifecycle with thread-safe locking
- **Context manager**: `AppStoreConnectClient` supports `with client:` pattern
- **Concurrent API calls**: `ThreadPoolExecutor` for price-point batches and subscription price setting. Territory price grids are batched 8 territories per request (the endpoint serves 8000 rows a page), so 175 territories cost ~22 calls, not 175
- **No vendor SDK**: the LLM call is a plain httpx POST in `llm.py`, so any OpenAI-compatible endpoint works (OpenRouter, Groq, Ollama, vLLM) and nothing in the dependency tree needs compiling. Retries live there — 429/5xx/timeouts, three attempts, 1s then 2s backoff; 4xx fails immediately
- **AI caching**: Results cached in `~/.cache/ppp-pricing/` (`$XDG_CACHE_HOME` honoured) by SHA-256 hash of app name; `--clear-cache` removes all cached results
- **Config discovery**: `--config` → `$PPP_PRICING_CONFIG` → nearest ancestor of cwd holding a `.env` → `~/.config/ppp-pricing/`. The first two are authoritative: a wrong explicit path fails loudly instead of silently falling back. `load_dotenv` is only ever called with an explicit path — never with `None`, which would make python-dotenv search upwards and override the chosen directory
- **IAP vs Subscription**: Different API endpoints and flows — IAPs use single atomic request, subscriptions need per-territory POST + pending price cleanup

## How to Run

```bash
pip install -e .
pytest                    # unit tests (173 tests)
pytest tests/integration  # integration tests (need real API keys)
ppp-pricing --app-id ID --iap PRODUCT_ID --dry-run
ppp-pricing --app-id ID --iap PRODUCT_ID --preserved --start-date 2026-08-01  # subscriptions only
ppp-pricing --clear-cache            # clear AI analysis cache
```

## Environment Variables

Required:
- `ASC_KEY_ID` — App Store Connect API Key ID
- `ASC_ISSUER_ID` — App Store Connect Issuer ID
- `ASC_PRIVATE_KEY_PATH` — Path to .p8 private key file

Optional:
- `LLM_API_KEY` / `OPENAI_API_KEY` — Enables AI pricing analysis
- `LLM_BASE_URL` / `OPENAI_BASE_URL` — Any OpenAI-compatible endpoint (OpenRouter, Groq, Ollama, vLLM); default `https://api.openai.com/v1`
- `LLM_MODEL` / `OPENAI_MODEL` — Model name; default `gpt-5.2`
- `LLM_REQUEST_TIMEOUT` / `OPENAI_REQUEST_TIMEOUT` — Seconds, default 120
- `ASC_REQUEST_TIMEOUT` — API timeout in seconds (default: 30)
- `PPP_PRICING_CONFIG` — Directory holding `.env` and the `.p8` key (same role as `--config`)
- `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` — Standard overrides for the config and cache directories

## Important Notes

- **GPT model is `gpt-5.2`** — this is correct, do not change it
- **USA is always excluded** from country list (it's the base price)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duceum/appstore-ppp-pricing-agent-skill](https://github.com/duceum/appstore-ppp-pricing-agent-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
