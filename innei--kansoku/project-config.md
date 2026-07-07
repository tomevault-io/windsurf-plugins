---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A personal **US-equities trading journal**, not a software product. The repo is three things:

1. **A durable record** — dated markdown under `journal/` and per-name notes under `stocks/`, plus chart data JSON under `journal/charts/data/`. These files are the *only* persistence layer (no database).
2. **A toolchain** — custom Claude Code skills under `.claude/skills/` that pull market data and orchestrate analysis workflows. "Running" this repo means invoking a skill or one of its Python scripts, then writing the synthesis back into a journal/stock file.
3. **A chart web app** — `app/` (pnpm workspace: `server/` Fastify + TS with Vite mounted in middleware mode — no build step, web edits hot-reload; `web/` Vite + React) renders all charts locally at `http://localhost:5199`. The server calls the longbridge CLI itself and computes every indicator in TS; charts are created via `POST /api/charts` (see `.claude/skills/chart/SKILL.md`). Realtime layer: SSE streams push live quotes (watchlist ∪ positions, pre/post/overnight aware) and 60s chart rebuilds while a page is open — persisted chart JSON stays frozen at analysis time. Start with `cd app && pnpm start`; tests with `cd app && pnpm test`.

**Documentation language — write every document in this repo in 中文白话 (modern vernacular Chinese).** This covers journal entries, stock notes, specs, READMEs, and this file. Keep English only for tickers, API/CLI identifiers, and terms with no natural translation. This **overrides** the global "products committed to git are written in English" default (`~/.claude/CLAUDE.md`) — for this repo, written docs are 中文白话, not English and not 文言.

**对话回复也用 中文白话，不用文言。** This project overrides the global 文言 chat-reply rule (`~/.claude/CLAUDE.md`). Every reply to the user — explanations, status updates, end-of-turn summaries — is plain modern Chinese.

**少用专业术语和英文行话。** The user is a retail investor, not a finance pro. Avoid jargon (Greeks, sharpe, drawdown, beta, alpha, basis point, hedge, IV, theta, gamma, basis, carry, skew, convexity, duration, P/E expansion, multiple compression, etc.) and English finance terms when a plain phrase works. **If a term truly has no plain equivalent, write it then immediately add a short bracketed gloss in 中文白话.** Examples:
- bad: "今天 SMH 的 IV crush 比较明显"
- good: "今天 SMH 的 IV crush（财报后期权隐含波动率坍缩，简单讲就是期权价格急跌）比较明显"
- bad: "回调到 50 日均线找支撑"
- good: "回调到 50 日均线（最近 50 个交易日的平均价，常被视为中期支撑）找支撑"

Tickers (NVDA / MRVL / SMH …), CLI / API names (`longbridge`, `fred`), and file paths stay in English without gloss — those are identifiers, not jargon.

**持仓相关的事情不要问用户，直接查长桥。** When you need to know what the user holds, position size, cost basis, P&L, or account balance, **do not ask** — invoke the `longbridge-positions` / `longbridge-profit-analysis` / `longbridge-portfolio` skill (or the `longbridge` CLI directly) and read the live account. The user finds the question annoying because the answer is already in the broker. Only ask if the broker call fails or returns ambiguous data.

**US markets only** — never query HK / CN / SG symbols in market-wide work.

## Architecture — three layers

### Layer 1 — data sources (raw retrieval)

| Source | Access | Covers |
|---|---|---|
| **Longbridge** plugin (`longbridge ...` CLI / `longbridge-*` skills) | brokerage account | real-time quotes, K-line/OHLCV, fundamentals, capital flow, technicals, market temperature, news |
| **`fred`** skill | free API key | US/global macro time series (CPI, GDP, Fed funds, yields, M2, DXY) |
| **`sec-edgar`** skill | UA header | raw 10-K/10-Q/8-K/S-1 text, Form 4 insider parsing |
| **`gdelt`** skill | none (5s throttle) | global multilingual news tone stream |
| **`trump-truth-monitor`** skill | RSS mirror | Trump Truth Social feed, classified + tier-graded for market impact |

Longbridge covers price/fundamentals; the four custom skills cover Longbridge's blind spots (macro, raw filings, world news, policy speech). See `docs/superpowers/specs/2026-05-28-market-intel-skills-design.md` for the design rationale and full per-script interface.

### Layer 2 — orchestration workflows (the value-add)

These skills do not fetch new kinds of data; they sequence Layer-1 calls into a disciplined read and enforce anti-patterns:

- **`stock-deep-dive`** — one-pass six-lens onboarding for a name you don't know (business / fundamentals / technicals / catalysts / supply-chain-peers / audit). Dispatch lenses 1–5 in **one parallel tool block** (8–12 `longbridge` calls); lens 6 audits the result.
- **`capital-rotation`** — one-shot end-of-session scan of net flows across fixed cohorts (indices / semis / software-cloud / mega-tech), names ONE rotation narrative, writes `journal/YYYY-MM-DD-flow.md`.
- **`market-session-tracker`** — live intraday monitoring of a watchlist across pre-market → close, with breakout verification, distribution detection, tier classification, and timestamped thesis revision.

**Routing (these three overlap — pick deliberately):**
- Single name, first look, multiple dimensions → `stock-deep-dive`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Innei/kansoku](https://github.com/Innei/kansoku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
