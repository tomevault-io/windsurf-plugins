---
trigger: always_on
description: Instructions for Codex when working on this codebase.
---

# AGENTS.md

Instructions for Codex when working on this codebase.

## Product Identity

**Polily — A Polymarket Monitoring Agent That Actually Works**

Finds structure, surfaces risk, sizes friction, watches positions. The user pulls the trigger today. Autopilot is on the roadmap but not the current default — design new features so they remain compatible with both modes.

## Design Context

Not a user-profile spec — these are facts about how the product is used, useful when deciding trade-offs:

- **Friction transparency over net numbers.** Polymarket's UI hides spread, depth, and fees — users get burned because the friction is invisible. Surface every fee / slippage / depth-constraint component explicitly; never silently absorb costs into a "net" number. The dollar amount changes with account size; the invisibility risk is the same at $50 and $20,000.
- **Manual operation, not automation.** Users click buttons; sub-second latency isn't the pressure. But signals must be scannable — a user should decide "look closer vs. skip" in ≤5s.
- **Daily review cadence.** ≥5s refresh intervals are fine; no tick-level streaming UI needed. Heartbeat-driven refresh (see `polily/tui/screens/main.py::_bus_heartbeat`) is the canonical pattern.
- **URL-driven depth, not scan breadth.** Users bring their own events. The pipeline is deep due-diligence on one event at a time, not shallow breadth across thousands. If a feature requires iterating 8000+ markets, it's likely a misunderstanding of the product shape.
- **Due diligence, not signal generation.** Output is "here's what the numbers say, here's the risk, here's the friction" — conditional framing, never unconditional commands like "buy YES".

## Architecture (Event-First)

**Data model:** Events (parent) → Markets (children). All state in unified SQLite (`data/polily.db`). No scan archives, no JSON files. Events carry tier/score/monitor state; markets carry prices/orderbook.

**Entry flow:** **URL-driven, single-event.** User pastes a Polymarket event URL into the TUI. `polily.url_parser` extracts the event slug. `polily.scan.pipeline.fetch_and_score_event` fetches the event + child markets from Gamma API → applies hard filters → computes structure score (5-dim) → runs mispricing detection → optionally calls NarrativeWriter agent → assigns tier → persists to events/markets tables. **There is no batch scan over 8000+ markets** — that pattern was removed in v0.5.0.

**Poll architecture:** Single global poll job runs every **30 seconds** on a dedicated 1-thread executor. Fetches prices for all markets the user has added to monitoring. Movement detection runs inline per tick (magnitude + quality scoring). If significant movement detected, triggers AI analysis on the `ai` executor (5 threads).

**Daemon:** Dual-executor APScheduler daemon (`polily/daemon/scheduler.py`). Poll executor (1 thread) for price polling. AI executor (5 threads) for concurrent analysis jobs. Managed via `polily scheduler run/stop/restart/status`. Started via launchd in production.

**Why "monitoring agent" rather than research assistant / signal generator?**
Users don't want data dumps (research) or commands (signals). They want something that keeps watching on their behalf and surfaces what actually changed — price moves, structure shifts, position risk, end-dates coming up. Conditional advice ("if you're bullish, this may have edge") is OK. Definitive signals are not.

**Why Structure Score ≠ trade quality?**
Score measures tradability (spread, depth, objectivity, time, friction). Not profitability. Always communicate it that way to users.

**Why URL-driven instead of batch scanning?**
Users have domain edge in specific events — they already know what they want to look at. Batch scanning produced noise and rate-limit issues; deep single-event analysis is what actually informs decisions.

**Why Binance (ccxt) not CoinGecko?**
CoinGecko free tier rate-limits aggressively (429 errors). Binance via ccxt has 6000 weight/min, no API key needed, first-party exchange data.

**Why AI agents use `Codex -p` CLI?**
Included in Codex subscription, no per-token cost. Response parsed from `result` field (not `structured_output`). JSON extracted from markdown code blocks via regex fallback.

## Coding Conventions

- Python 3.11+, type hints everywhere
- Pydantic for data models and config
- `async` for I/O (HTTP, ccxt), `sync` for pipeline orchestration
- TDD: write test first (red), implement (green), refactor
- Chinese for all user-facing output (terminal, narratives, prompts)
- English for code, variable names, comments
- No unnecessary abstractions — three similar lines beats a premature abstraction
- Config-driven: thresholds, weights, behavior live in `polily.db` `config` table (canonical since v0.10.0). TUI ⚙ 配置 edits live values via `save_knob`; `config.yaml` on disk is a read-only auto-generated snapshot regenerated after each save. Do NOT edit `config.yaml` and expect it to load — it's overwritten on every TUI launch / save / `polily config reset`. The CLI escape hatch is `polily config reset --all` or `polily config reset <key>`. Pre-v0.10.0 `config.yaml` files are auto-migrated to db on first boot (invalid yaml is preserved as `config.yaml.bak`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShiyuCheng2018/polily](https://github.com/ShiyuCheng2018/polily) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
