---
trigger: always_on
description: > **Role:** This document defines the project context and operational constraints for AI Agents and automated contributors. Adherence is mandatory to ensure project stability and maintainability.
---

# CLAUDE.md — Project Context for AI Assistants

> **Role:** This document defines the project context and operational constraints for AI Agents and automated contributors. Adherence is mandatory to ensure project stability and maintainability.

---

## Project
**Cache Me If You Can** — START Hack 2026
Team: Constantin Salzer · Lukas Kapferer · Jamie Maier · Dorian Markies

## Tech Stack
- **Backend:** Python · FastAPI (`backend/main.py`) — run with `uvicorn backend.main:app --reload`
- **Frontend:** TypeScript · Next.js + Tailwind CSS + Framer Motion (`frontend/`) — run with `cd frontend && npm run dev`
- **State Management:** Zustand
- **Charts:** TradingView Lightweight Charts
- **Battle Graphics:** Phaser 3 (WebGL, procedural knights, tween animations, particle FX)
- **Database:** Supabase (PostgreSQL) — client in `db/database.py`
- **AI:** Anthropic Claude API (educational insights)
- **Real-time:** FastAPI WebSockets (battle mode)
- **Deployment:** Vercel (frontend) · Railway (backend) · Supabase (DB)

## Project Structure
```
STARTHACK26/
├── backend/
│   ├── main.py              # FastAPI routes, endpoints, WebSocket, CORS
│   ├── auth.py              # Authentication helpers
│   ├── battle.py            # WebSocket battle room state machine
│   ├── ticker.py            # Ticker / market data helpers
│   └── data/
│       ├── historical.py    # Historical market data + simulate_portfolio()
│       ├── gbm.py           # Geometric Brownian Motion simulation
│       └── *.csv            # Market data (Bonds, Equities, FX, Gold, SMI, DJIA)
├── frontend/                # Next.js App Router
│   ├── src/app/             # Pages: / (landing), /sandbox, /battle, /battle/[roomId]
│   │   └── (shell)/         # Authenticated shell: sandbox/, battle/, clans/, dashboard/, review/
│   ├── src/components/      # Reusable: portfolio-builder, asset-card, charts, ai-insight, sidebar, …
│   ├── src/lib/             # api.ts, ws.ts, constants.ts, types.ts
│   └── src/store/           # Zustand game store
├── db/database.py           # Supabase client (import db from here)
├── .env                     # API keys — never commit!
├── .env.example             # Key template
└── requirements.txt
```

## Game Flow
```
Landing Page → Sandbox Mode → Battle Mode
     |              |              |
  Username    Learn assets,   Real-time 1v1,
  entry       build & test    historical scenario,
              portfolios      AI insight
```

## Environment Variables
All keys live in `.env` (see `.env.example`):
- `ANTHROPIC_API_KEY` — console.anthropic.com
- `SUPABASE_URL` / `SUPABASE_KEY` — supabase.com → project settings → API
- `FRONTEND_URL` — for CORS (e.g., `http://localhost:3000` or Vercel URL)

Frontend env vars live in `frontend/.env.local`:
- `NEXT_PUBLIC_API_URL` — backend URL (e.g., `http://localhost:8000`)
- `NEXT_PUBLIC_WS_URL` — WebSocket URL (e.g., `ws://localhost:8000`)

## Team Roles
- **Constantin** — Lead Engineer · Backend · API Architecture
- **Lukas** — ML / Data Lead · Integration
- **Jamie** — Quant Analytics · Frontend · Data Visualization
- **Dorian** — Business Strategy · Pitch

---

## Best Practices

* **Linguistic Uniformity:** The project's primary and only language is **English**. All technical documentation, directory structures, file names, and internal communications must be in English.
* **Content Integrity:** Never alter prose, marketing copy, or messaging unless explicitly instructed.
* **Backward Compatibility:** All changes must be non-breaking. Flag potential breaking changes for manual review before implementation.
* **Decoupling:** Maintain a strict separation between **Logic** (functions), **Layout** (structure), and **Styling** (CSS/Tokens).
* **No Hardcoding:** Content values must be passed via props, state, or schemas. Never embed raw strings inside UI components. Never hardcode secrets — always use environment variables.
* **Visual Stability:** Ensure changes do not introduce Cumulative Layout Shift (CLS) or rendering flickers during hydration.
* **Performance:** Ensure changes do not introduce performance regressions, latency, input lag, or any other performance-hindering measurements.
* **Verification:** Visually verify implementations for both **Desktop** and **Mobile** breakpoints.

---

## Commit Standard

Agents must provide a descriptive commit header at the end of every response. **Do not attempt to execute the commit/push or stage files yourself.**

**Format:**
`[type|scope] short, precise description`

| Type | Description |
| :--- | :--- |
| `feat` | A new feature or capability |
| `fix` | A bug fix |
| `refactor` | Structural improvement (e.g., optimizing the hot path) |
| `style` | Formatting or visual adjustments |
| `docs` | Documentation updates |
| `perf` | Critical performance/latency improvements |
| `chore` | Maintenance, dependencies, or build tasks |

**Examples:**
* `[perf|engine] reduce memory allocation in order-matching loop`
* `[fix|api] resolve websocket reconnection race condition`
* `[refactor|schema] optimize data alignment for cache efficiency`
* `[feat|monitor] add microsecond-level latency tracking`

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Luggias) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-12 -->
