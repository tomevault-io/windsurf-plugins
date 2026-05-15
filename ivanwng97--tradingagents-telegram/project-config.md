---
trigger: always_on
description: Telegram bot wrapping the [TradingAgents](https://github.com/TauricResearch/TradingAgents) library. Users curate a watchlist via Telegram, tap a ticker, and the bot runs `TradingAgentsGraph.propagate(...)` and posts a finviz chart + Telegraph link with the verdict. Per-step pipeline progress is streamed back into the message caption while the analysis runs.
---

# TradingAgents-Telegram — Architecture Reference

Telegram bot wrapping the [TradingAgents](https://github.com/TauricResearch/TradingAgents) library. Users curate a watchlist via Telegram, tap a ticker, and the bot runs `TradingAgentsGraph.propagate(...)` and posts a finviz chart + Telegraph link with the verdict. Per-step pipeline progress is streamed back into the message caption while the analysis runs.

## Layout

`src/tg_bot/` — package. Entry points: `app.py` (PTB Application + `BOT_COMMANDS` + lifecycle hooks), `__main__.py` (`python -m tg_bot`). Core flow modules: `analysis.py` (`run_trading_analysis` + `GraphPool` + model catalog), `cache.py` (same-day result cache, keyed by `AnalysisConfigKey`), `config_key.py` (`AnalysisConfigKey` — single source of truth for the (provider, deep, quick, rounds, effort) tuple shared by cache slug, caption "via" line, and Telegraph title), `progress.py` (cancel-aware LangChain callback), `digest.py` (`/digest` picker), `validation.py` (yfinance-backed ticker check + shared `TICKER_RE`), `history.py` (disk-reader for past analyses; imports `TICKER_RE` from validation), `chart.py` (finviz URL), `formatters.py` (HTML + MarkdownV2 captions — see Invariant #4), `telegraph_client.py` (Telegraph publish), `auth.py` (authorize TypeHandler), `config.py` (env-driven `Config`). Handlers: `handlers/commands.py` (slash commands) + `handlers/callbacks.py` (prefix-dispatched inline buttons). Storage: `storage/{_base,watchlist,user_config}.py` — `JsonStorage` base + two singletons re-exported from `storage/__init__.py`.

Top-level: `pyproject.toml`, `Dockerfile`, `docker-compose.yml`, `.env`, `data/` (runtime state, gitignored), `docs/` (DEVELOPMENT / TROUBLESHOOTING / CONFIGURATION / MANUAL_INSTALL / TODO), `.github/workflows/` (lint + Docker build + CodeQL + on-demand Claude review).

## Architecture (for code reviewers)

This section gives reviewers — human or LLM — the structural context needed to spot cross-file regressions before "Key contracts" goes deep on each subsystem.

### Request lifecycle (manual `/watch` tap)

1. Telegram delivers Update → PTB queue. `concurrent_updates=True` ensures cancel taps aren't queued behind in-flight analyses.
2. **Auth gate** (`auth.py:authorize`, group=-1) — drops Update if user not in `ALLOWED_USER_IDS`. Fail-closed when `effective_user` is missing and an allowlist is set.
3. **Handler dispatch** — commands match by name; callbacks dispatch by prefix (order-sensitive where overlapping: `cancel_analysis:` before `cancel:`, `digest_cancel:` before `digest:`).
4. Picker accumulates selection in `chat_data["watch_selection"]`; Done routes through `_handle_done` → `_run_analysis_for_ticker` per ticker.
5. **Cache lookup** (`cache.lookup(key, ticker, date_iso)`) where `key = AnalysisConfigKey.from_config(config)` — hits short-circuit before semaphore acquire, reuse persisted `telegraph_url`, skip the progress flow.
6. **Cancel registry** — write `chat_data["analysis_cancels"][run_id] = {"event": threading.Event, "async_event": asyncio.Event, "message_id": None}` on entry (`message_id` filled after `send_photo`); pop in `finally`. Cancel callback is `cancel_analysis:<run_id>` (UUID, not message_id).
7. **Concurrency gate** — `asyncio.wait([sem.acquire(), cancel_async.wait()])` races slot acquisition vs queued-cancel. Above-cap users see `⏳ Queued`.
8. **GraphPool acquire** — keyed on `(provider, deep, quick, rounds, effort)`; per-key cap matches the semaphore so the pool's blocking-queue branch is unreachable.
9. **`to_thread(propagate)`** — LangGraph runs; tradingagents threads our `BaseCallbackHandler` into LLM kwargs.
10. **Per-step progress** — `on_chat_model_start` → `ProgressReporter._dispatch` → `editMessageCaption` (re-attaches cancel keyboard; Telegram drops `reply_markup` otherwise). `cancel_event` is checked **before** every step; raising `CancelledByUserError` aborts the in-flight LLM call (requires `raise_error=True` on the handler — LangChain swallows handler exceptions by default).
11. **Race-close checks** — three: post-`to_thread`, post-Telegraph publish, before final caption edit. A late tap discards instead of overwriting.
12. **Telegraph publish** via `to_thread(edit_page)` if `/refresh` (uses cached URL's path), else `to_thread(create_page)`. Title comes from `AnalysisConfigKey.telegraph_title(ticker)` so the URL embeds the config.
13. **Cache store** (`cache.store(key, ticker, date, ...)`) — full `final_state` (LangChain messages coerced via `_json_default`); atomic write (tempfile + fsync + rename); stale-date siblings swept *after* the rename succeeds. The store no-ops when `telegraph_url` is falsy (cache-hygiene gate at the write site).
14. **Final caption edit** + pool release + cancel registry pop.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IvanWng97/TradingAgents-Telegram](https://github.com/IvanWng97/TradingAgents-Telegram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
