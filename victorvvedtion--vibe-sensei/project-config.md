---
trigger: always_on
description: This file provides guidance to AI assistants working with code in this repository.
---

# CLAUDE.md

This file provides guidance to AI assistants working with code in this repository.

## Project Overview

**Vibe Sensei** is an AI trading terminal with 68 master guardians. Historical trading legends, philosophers, and scientists act as your personal risk guardians. Each user is deterministically assigned a master who watches trades, warns in character, and debates other masters before big moves. Includes a paper trading sandbox (100k USDT starting balance) powered by CCXT.

- **Runtime:** Bun
- **UI:** React + Ink (terminal rendering)
- **Charts:** TradingView Lightweight Charts + UDF data server
- **Exchange:** CCXT (paper mode default, live mode supported)

## Commands

```bash
bun install          # Install dependencies
bun run dev          # Interactive REPL
# bun run dev -- --web # REPL + TradingView chart server (:3456) — web端暂停开发
bun run build        # Production single-file bundle to dist/cli.js
```

No test runner is configured. No linter is configured.

## Architecture

### Entry & Bootstrap

1. **`src/entrypoints/cli.tsx`** — True entrypoint. Sets up runtime globals and polyfills.
2. **`src/main.tsx`** — Commander.js CLI definition. Parses args, initializes services, launches the REPL or runs in pipe mode.
3. **`src/entrypoints/init.ts`** — One-time initialization (config, trust dialog).

### Core Loop

- **`src/query.ts`** — Main query function. Sends messages to the AI API, handles streaming responses, processes tool calls, manages the conversation turn loop, and runs the guardian observer after trading tool calls.
- **`src/QueryEngine.ts`** — Higher-level orchestrator wrapping `query()`. Manages conversation state, compaction, file history snapshots, and turn-level bookkeeping.
- **`src/screens/REPL.tsx`** — Interactive REPL screen (React/Ink component). Handles user input, message display, tool permission prompts, and keyboard shortcuts.

### Guardian System (`src/buddy/`)

The core personality and risk layer of Vibe Sensei.

- **`types.ts`** — Master roster (68 masters), rarities (Legendary/Epic/Rare/Uncommon/Common), quotes, stat definitions.
- **`guardian.ts`** — `RiskGuardian` class. Evaluates position size and drawdown after every trade tool call.
- **`persona.ts`** — Persona engine: 9 archetypes, 5 stat dimensions (PRECISION, PATIENCE, AGGRESSION, WISDOM, SASS), tone modifiers.
- **`companion.ts`** — Deterministic master assignment via `mulberry32(hash(userId))`.
- **`debate.ts`** — Adversarial debates: two masters argue for/against before big trades.
- **`consultation.ts`** — Cross-guardian second opinions from any of the 68 masters.
- **`ghost-warnings.ts`** — Cautionary apparitions from crypto's fallen (SBF, Do Kwon, 3AC, Newton).
- **`diary.ts`** — Evolution diary: guardian learns user trading habits over time.
- **`trade-card.ts`** — Shareable text trade cards for Twitter/X with box-drawing art.
- **`checks/`** — Risk check implementations: `position_size`, `drawdown`.

### Trading Tools (`src/tools/`)

- **`OrderTool/`** — `PlaceOrder`: market, limit, and stop-loss orders.
- **`PositionTool/`** — `GetPositions`: view open positions with PnL.
- **`BalanceTool/`** — `GetBalance`: check portfolio balance across assets.
- Additional tools: `BashTool`, `FileEditTool`, `GrepTool`, `AgentTool`, and others in `src/tools/<ToolName>/` directories.
- Tools define: `name`, `description`, `inputSchema` (JSON Schema), `call()` (execution), and optionally a React component for rendering results.

### Exchange Layer (`src/services/exchange/`)

- **`singleton.ts`** — Exchange singleton: shared CCXT instance across all trading tools.
- Paper trading mode by default (safe sandbox with simulated fills). Live mode supported via configuration.

### Chart System

- **`src/services/chart/`** — UDF (Universal Data Feed) server for TradingView chart data.
- **`web/`** — TradingView Lightweight Charts frontend, served on `:3456` when `--web` flag is used.

### Guardian Observer (`src/services/trading/guardian-observer.ts`)

Hooks into the query loop. After every trading tool call, the guardian observer triggers the `RiskGuardian` to evaluate position size and drawdown checks. Alerts are delivered in the assigned master's voice and personality.

### UI Layer

- **`src/screens/REPL.tsx`** — Main interactive terminal screen.
- **`src/components/`** — React/Ink terminal UI components: message rendering, prompt input, permission prompts, trade displays.
- **`src/ink/`** — Internal Ink framework: custom reconciler, hooks (`useInput`, `useTerminalSize`), virtual list rendering.

### State Management

- **`src/state/AppState.tsx`** — Central app state type and context provider.
- **`src/state/store.ts`** — Store for AppState.
- **`src/bootstrap/state.ts`** — Module-level singletons for session-global state (session ID, CWD, project root, token counts).

### API & Providers (`src/services/api/providers/`)

Multi-provider layer. All providers emit Anthropic-shaped stream events so the REPL renders them identically.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VictorVVedtion/vibe-sensei](https://github.com/VictorVVedtion/vibe-sensei) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
