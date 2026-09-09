---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Directional Scalper is an algorithmic crypto trading framework built on top of [CCXT](https://github.com/ccxt/ccxt). It runs grid/hedging market-making strategies across multiple perpetual-futures exchanges (Bybit is the primary, first-class venue). `config.py` reports `VERSION = "v2.9.9"`. Python 3.11 only.

## Running the bot

The **primary, actively-developed entry point is `multi_bot_aio.py`** — an all-in-one multi-symbol bot that auto-rotates through symbols and manages one thread per symbol.

```bash
python3.11 multi_bot_aio.py --exchange bybit --account_name account_1 --strategy qsgridob --config configs/config.json
```

- Missing `--exchange` / `--strategy` / `--account_name` are prompted interactively via `inquirer`.
- `--config` defaults to `configs/config.json`, but the `__main__` block **always** loads credentials from the hardcoded path `configs/account.json` (the `--config` file supplies everything else).
- Only a small set of strategy names are wired into the live dispatcher (`run_strategy` in `multi_bot_aio.py`): `qsgridob` and `qsgridob_nosignal` (both → `LinearGridBaseFutures`) and `qstrendobdynamictp`. The much larger strategy library under `core/strategies/` is mostly legacy/unwired.

`bot.py` is the **legacy single-symbol** runner. It imports per-exchange single strategies (Bitget/OKX/Bybit/Huobi/Binance/Phemex/MEXC) and is what Docker and the tmuxp scripts drive:

```bash
docker-compose run directional-scalper python3.11 bot.py --symbol SUIUSDT --strategy bybit_hedge_mfirsi_maker --config config_main.json
```

`multi_bot_signalscreener_targetcoin.py` is a signal-screening variant of the multi-bot. `api/scraper.py` and `api/multiprocessing_api*.py` are standalone data scrapers that produce the volume/spread JSON served from quantumvoid.org.

### Process supervision (production)
`start_tmuxp.sh` loads `bybit.yaml` (a tmuxp session that launches many `bot.py` panes); `check_tmuxp.sh` is a cron-style watchdog that restarts the session if fewer than N `bot.py` processes are running. Both hardcode `/opt/directionalscalper`.

## Setup, lint, test

```bash
pip install -r requirements.txt          # runtime deps (used by Dockerfile)
pipenv install --dev                     # dev environment with linters
pre-commit install                       # install git hooks
pre-commit run --all-files               # run all linters manually
pytest -vv                               # pytest.ini sets pythonpath=.
```

- **Linters (via pre-commit):** black (line-length 88), isort (black profile), flake8 (max-line 88, max-complexity 18, ignores E203/E266/E501/W503/F403/F401/E402), mypy, bandit. Match black formatting on any code you touch.
- **Tests:** `pytest.ini` exists but there are currently **no test files** in the repo. There is no build step (pure Python).
- **Dependency caveat:** `requirements.txt` pins `ccxt==4.4.52` (authoritative — Docker uses it) while `Pipfile` still pins `ccxt==3.0.78`. Prefer `requirements.txt`.

## Configuration model

Configuration is **split across two JSON files** and merged at load time by `load_config()` in `config.py`:

- `configs/config.json` — bot parameters, strategy settings, and a list of `exchanges` each with `name`, `account_name`, and `symbols_allowed`.
- `configs/account.json` — API keys, matched into the config by the `(name, account_name)` pair.

Copy `configs/config_example.json` and `configs/account_example.json` to create these. The whole structure is validated by Pydantic models in `config.py` (`Config`, `Bot`, `Exchange`, `API`, `Discord`/`Telegram`, `Hotkeys`). Many bot-level params have `@validator` bounds (e.g. percentages must be 0.0–1.0) — adding a config field means adding it to the relevant Pydantic model or load will fail.

The grid strategy is configured almost entirely through the nested `bot.linear_grid` dict (levels, wallet exposure, auto-hedge, stop-loss, sticky-size, etc.). This dict is required (`linear_grid` validator rejects `None`).

## Architecture

Three layers, each with a base class extended per-exchange / per-strategy:

**Exchange layer** — `directionalscalper/core/exchanges/`
- `Exchange` (`exchange.py`) is the CCXT-wrapping base class. Per-venue subclasses (`BybitExchange`, `BinanceExchange`, `BitgetExchange`, `HyperLiquidExchange`, `BlofinExchange`, `MexcExchange`, `HuobiExchange`, `LBankExchange`) live alongside it and are exported from `exchanges/__init__.py`.
- Exchange-specific behavior is exposed as suffixed methods (e.g. `get_balance_bybit`, `get_all_open_positions_bybit`) and frequently called **dynamically via `getattr(exchange, f"...{exchange_name}")`** in the entry points. When adding an exchange method, follow the `_<exchangename>` naming convention so the dynamic dispatch finds it.

**Strategy layer** — `directionalscalper/core/strategies/`
- Inheritance chain: `BaseStrategy` (`base_strategy.py`, ~5.5k lines; also defines `OrderBookAnalyzer`) → `BybitStrategy` (`bybit/bybit_strategy.py`, ~18k lines, the bulk of trading logic) → concrete strategies like `LinearGridBaseFutures` (`bybit/gridbased/lineargrid_base.py`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [donewiththedollar/directionalscalper](https://github.com/donewiththedollar/directionalscalper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
