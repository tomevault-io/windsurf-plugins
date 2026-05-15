---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**NofyAI** is a Next.js 16 web dashboard for an AI-powered algorithmic trading system. The system supports multi-trader competitions where different AI models (DeepSeek, Qwen, custom) trade independently on cryptocurrency exchanges (Aster DEX, Binance, Hyperliquid). The frontend displays real-time performance tracking, AI decision transparency, and comprehensive analytics.

## Common Commands

```bash
# Development
npm run dev          # Start Next.js dev server with Turbopack (http://localhost:3000)
npm run build        # Build for production
npm start            # Start production server
npm run lint         # Run ESLint

# Utilities
npx tsx scripts/test-aster-connection.ts   # Test Aster DEX connection
npx tsx scripts/derive-address.ts          # Derive wallet address from private key
```

## Architecture Overview

### In-Process Trading Engine

**IMPORTANT**: Unlike what the README suggests, this is NOT a client-server architecture with a separate Go backend. The trading engine runs **in-process** within the Next.js application itself.

- **TraderManager** (`/lib/trader-manager.ts`): Singleton that manages multiple `TradingEngine` instances
- **TradingEngine** (`/lib/trading-engine.ts`): Core trading logic for each AI trader
- **DecisionLogger** (`/lib/decision-logger.ts`): Persists trading decisions to `decision_logs/{trader_id}/` directory
- **API Routes** (`/app/api/*`): Next.js route handlers that call TraderManager methods directly (not HTTP proxy)

### Singleton Pattern for Hot-Reload Safety

The codebase uses `globalThis` to persist singletons across Next.js hot reloads in development:

```typescript
// Pattern used in trader-manager.ts, config-loader.ts
const globalForTraderManager = globalThis as unknown as {
  traderManager: TraderManager | undefined;
};

export async function getTraderManager(): Promise<TraderManager> {
  if (!globalForTraderManager.traderManager) {
    // Initialize once
  }
  return globalForTraderManager.traderManager;
}
```

**Why this matters**: Without this pattern, hot reloads would create duplicate trading engines and multiple trading sessions.

### Multi-Trader Architecture

Each trader operates independently:
- Separate configuration in `config.json`
- Independent trading sessions (can start/stop individually)
- Isolated decision logs in `decision_logs/{trader_id}/`
- Own AI model and exchange credentials

### Configuration System

**Setup**: Copy `config.json.example` to `config.json` and configure:

```json
{
  "traders": [
    {
      "id": "aster_deepseek",        // Unique trader identifier
      "name": "Aster DeepSeek Trader",
      "enabled": true,                // Must be true to auto-start
      "ai_model": "deepseek",         // "deepseek" | "qwen" | "custom"
      "exchange": "aster",            // "aster" | "binance" | "hyperliquid"
      // Exchange credentials
      "aster_user": "0x...",
      "aster_signer": "0x...",
      "aster_private_key": "...",
      // AI credentials
      "deepseek_api_key": "sk-...",
      // Trading params
      "initial_balance": 1000.0,
      "scan_interval_minutes": 3
    }
  ],
  "leverage": {
    "btc_eth_leverage": 5,
    "altcoin_leverage": 5
  }
}
```

**ConfigLoader** (`/lib/config-loader.ts`): Loads and validates `config.json`, masks sensitive keys for frontend display.

## Data Flow

### API Request Flow

```
Frontend Component
  → SWR hook (`useSWR('/api/account?trader_id=...')`)
  → Next.js API Route (`/app/api/account/route.ts`)
  → TraderManager.getTrader(traderId)
  → TradingEngine.getCurrentAccount()
  → AsterTrader.getBalance() / BinanceTrader / etc.
  → Exchange API
```

**No external backend**: All API routes call in-process methods.

### Trading Cycle Flow

```
TradingEngine.runCycle()
  → Get account balance & positions from exchange
  → Fetch market data for candidate coins
  → Build TradingContext (account, positions, market data)
  → Call AI model (getFullDecision in /lib/ai.ts)
  → Parse AI decisions (open/close/hold)
  → Execute trades via AsterTrader/BinanceTrader
  → Log decision + results to DecisionLogger
```

### Decision Logging

Each trading cycle creates a `DecisionRecord` saved to:
```
decision_logs/{trader_id}/{cycle_number}.json
```

Contains:
- AI chain-of-thought reasoning (`cot_trace`)
- Decisions array (symbol, action, reasoning)
- Account snapshot (before execution)
- Positions snapshot (before execution)
- Execution results (success/error per decision)
- Full input prompt sent to AI
- Candidate coins analyzed

**Performance Metrics**: Calculated by analyzing closed positions from decision logs (not real-time).

## Key Implementation Details

### DecisionDetailModal Merging Logic

**Location**: `/components/trader/DecisionDetailModal.tsx`

Displays AI trading decisions by merging two arrays:
- `decisions[]` - AI's intended actions
- `execution_results[]` - Actual execution outcomes

**Important**: The modal must:
1. Match decisions to execution results by symbol + action
2. Find corresponding position from `positions_snapshot` for each decision

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [osbrain/nofyai](https://github.com/osbrain/nofyai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
