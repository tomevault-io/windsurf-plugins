---
trigger: always_on
description: > This file is the single source of truth for Claude Code working on this project.
---

# CSE AI Investment Intelligence Dashboard — CLAUDE.md

> This file is the single source of truth for Claude Code working on this project.
> Read it fully before writing any code. Re-read it at the start of every session.

---

## 🔴 ABSOLUTE RULES (Non-Negotiable)

1. **The `.env` file must NEVER be touched, modified, read, cat'd, echo'd, or deleted under ANY circumstances.** Use `process.env.*` to access values the app already loads. This rule overrides ALL other instructions.
2. **Never place orders, click buy/sell buttons, or modify account data on ATrad.** All ATrad automation is READ-ONLY (scrape holdings, balances, account data).
3. **Never run destructive database operations** (DROP, TRUNCATE, DELETE without WHERE) without explicit user approval.
4. **Never install npm packages without stating which ones and why first.**
5. **Never commit `.env`, `node_modules`, or personal financial data to git.**

---

## Project Overview

**What:** A personal AI-powered Shariah-compliant investment intelligence platform for the Colombo Stock Exchange (CSE).

**Why:** Increase Sri Lankan household stock market participation through accessible, AI-driven, Shariah-filtered analysis. Currently a personal tool — future public platform.

**Who:** Single user (Atheeque), conservative risk profile, LKR 10,000/month Rupee Cost Averaging strategy, Shariah-compliant only.

**Where:** Runs locally on WSL2 Ubuntu. Managed via PM2 for persistence. No cloud deployment yet.

---

## Tech Stack

| Layer | Technology | Port/Path |
|-------|-----------|-----------|
| Frontend | Next.js 14 + TypeScript + Tailwind + shadcn/ui | localhost:3000 |
| Backend | NestJS + TypeScript | localhost:3001 |
| Database | PostgreSQL 16 | localhost:5433 |
| Cache | Redis 7 | localhost:6379 |
| Browser Automation | Playwright (Chromium) | Headless |
| AI | Claude API (Haiku for digests, Sonnet for analysis) | api.anthropic.com |
| Charts | TradingView Lightweight Charts + Recharts | — |
| Process Manager | PM2 | — |

**Repo:** `~/workspace/cse-ai-dashboard/`
**GitHub:** `https://github.com/Azi023/cse-ai-dashboard.git`

---

## Architecture

```
src/
├── backend/          # NestJS API server
│   └── src/
│       └── modules/
│           ├── cse-data/        # CSE API polling, stock sync, market data
│           ├── ai-engine/       # Claude API integration, briefs, signals, chat
│           ├── portfolio/       # Holdings, P&L, manual add
│           ├── shariah/         # Shariah screening (Almas whitelist)
│           ├── notifications/   # Daily digest, weekly brief, alerts
│           ├── analysis/        # [TO BUILD] Scoring engine, recommendations
│           ├── atrad-sync/      # ATrad Playwright browser automation
│           ├── news/            # RSS feed aggregation
│           ├── journey/         # Investment journey tracking, KPIs
│           ├── dividends/       # Dividend tracking, purification
│           ├── alerts/          # Price alerts, notifications bell
│           ├── signals/         # AI-generated trading signals
│           └── macro/           # CBSL macro data
├── frontend/         # Next.js dashboard UI
│   └── src/app/
│       ├── dashboard/    # Market overview, AI brief
│       ├── stocks/       # Stock browser, detail pages
│       ├── portfolio/    # Holdings, P&L, ATrad sync
│       ├── journey/      # Investment journey, KPIs, goals
│       ├── signals/      # AI signals display
│       ├── alerts/       # Notifications
│       ├── chat/         # AI Strategy Chat
│       └── news/         # News intelligence
├── scripts/          # Standalone scripts (ATrad recon, data tools)
└── data/             # Generated data, AI outputs, ATrad sync dumps
    ├── ai-generated/
    ├── atrad-sync/
    └── tracking/     # KPI tracker (gitignored)
```

---

## Data Sources

| Source | Method | Frequency | Notes |
|--------|--------|-----------|-------|
| CSE Market Data | POST to cse.lk/api/* (22 endpoints) | 5 min during market hours | Reverse-engineered, no auth needed |
| ATrad (HNB Stockbrokers) | Playwright browser automation | On-demand via Sync button | READ-ONLY. Login → scrape holdings + balance |
| Almas Equities Whitelist | Manual / twice-weekly screening | Mon & Thu 9:00 AM | Shariah compliance source of truth |
| RSS News | Economy Next, Daily FT, Google News CSE | Every 30 min (8AM-8PM weekdays) | daily_ft feed frequently fails to parse |
| CBSL Macro | Manual Excel import | As released | OPR, inflation, FX reserves |
| Claude AI | API calls for briefs, signals, digests | Cached with TTLs | Haiku for digests, Sonnet for analysis |

---

## Cron Schedule (All times Sri Lanka Time, UTC+5:30)

| Time | Job | Days | What It Does |
|------|-----|------|-------------|
| 9:25 AM | preMarketWarmup | Mon-Fri | Fetch initial market data before open |
| 9:30-14:30 | pollMarketData | Mon-Fri | Every 5 min: 7 CSE endpoints + trade summary |
| 9:30-14:30 | pollAnnouncements | Mon-Fri | Every 15 min: financial + approved announcements |
| 8:00-20:00 | fetchNews | Mon-Fri | Every 30 min: RSS feeds |
| 9:00 AM | shariahScreening | Mon, Thu | Twice-weekly Shariah compliance check |
| 2:35 PM | postCloseSnapshot | Mon-Fri | Final market data + trade summary after close |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Azi023) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
