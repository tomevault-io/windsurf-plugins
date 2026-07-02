---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TradeX is a real-time cryptocurrency trading platform with live price feeds, candlestick charts, and order management. This is a Turborepo monorepo using Bun as the package manager, with multiple apps and shared packages.

## Development Commands

### Root-level Commands
- `bun install` - Install all dependencies across the monorepo
- `turbo dev` - Start all apps in development mode (parallel)
- `turbo build` - Build all apps
- `turbo lint` - Lint all apps
- `bun run format` - Format all files with Prettier

### Local Infrastructure
Start PostgreSQL and Redis (required before running any app):
```bash
docker compose up -d
```
MongoDB is **not** in the docker-compose file — it must be set up separately (used by the engine for snapshots).

### Database Commands (from packages/db)
- `cd packages/db && bunx prisma generate` - Generate Prisma Client
- `cd packages/db && bunx prisma migrate dev` - Run database migrations (development)
- `cd packages/db && bunx prisma migrate deploy` - Run database migrations (production)
- `cd packages/db && bunx prisma studio` - Open Prisma Studio GUI

### App-specific Commands

**Web (Frontend):**
```bash
cd apps/web
bun run dev        # Start Vite dev server (http://localhost:5173)
bun run build      # Build for production
bun run build:dev  # Build for development
bun run lint       # Run ESLint
```

**API (Backend):**
```bash
cd apps/api
bun run dev        # Start Express server with tsx watch (http://localhost:3000)
bun run build      # Compile TypeScript
bun run start      # Run compiled code
```

**Engine (Trading Logic):**
```bash
cd apps/engine
bun run dev        # Run with Bun watch mode
bun run start      # Run without watch
```

**Pooler (Price Feed):**
```bash
cd apps/pooler
bun run dev        # Run with Bun watch mode
bun run start      # Run without watch
```

**WS (WebSocket Server):**
```bash
cd apps/ws
bun run dev        # Start WebSocket server (ws://localhost:8080)
```

**Mobile (React Native/Expo):**
```bash
cd apps/mobile
bun run start      # Start Expo dev server
bun run android    # Start on Android
bun run ios        # Start on iOS
bun run lint       # Run ESLint
```

## Architecture

### High-Level Data Flow

1. **Price Discovery:**
   - `pooler` connects to Backpack Exchange WebSocket
   - Publishes price updates to Redis channel `ws:price:update` and Redis Stream `stream:engine`
   - Prices broadcast every 100ms

2. **Price Broadcasting:**
   - `ws` service subscribes to `ws:price:update` Redis channel
   - Broadcasts prices to all connected WebSocket clients

3. **Order Execution:**
   - `api` receives HTTP requests and pushes messages to Redis Stream `stream:engine`
   - `engine` consumes from Redis Stream, processes orders in-memory
   - `engine` checks stop-loss, take-profit, and liquidation conditions on every price update
   - `engine` acknowledges via Redis pub/sub channel `http:response`

4. **State Persistence:**
   - `engine` maintains in-memory state (`prices`, `users`) in `memoryDb.ts`
   - Snapshots saved to MongoDB every 15 seconds
   - On restart, `engine` restores snapshot and replays missed Redis Stream messages

### Service Communication

- **API → Engine:** Redis Streams (`stream:engine`)
- **Engine → API:** Redis Pub/Sub (`http:response:{requestId}`)
- **Pooler → WS:** Redis Pub/Sub (`ws:price:update`)
- **Pooler → Engine:** Redis Streams (`stream:engine`)
- **WS → Frontend/Mobile:** WebSocket connection

### Apps

**apps/api** - Express REST API
- Handles authentication (JWT + bcrypt)
- User creation, balance queries
- Order creation/closure requests
- Communicates with engine via Redis Streams and awaits acknowledgment via pub/sub

**apps/engine** - Trading Engine (Bun runtime)
- Consumes messages from Redis Stream `stream:engine`
- Maintains in-memory order book and user balances
- Processes price updates and triggers stop-loss/take-profit/liquidation
- Snapshots state to MongoDB every 15s for crash recovery
- Uses consumer groups for fault tolerance

**apps/pooler** - Price Feed Service (Bun runtime)
- Connects to Backpack Exchange WebSocket (`wss://ws.backpack.exchange/`)
- Subscribes to BTC_USDC, ETH_USDC, SOL_USDC book tickers
- Publishes to Redis channel `ws:price:update` and Redis Stream `stream:engine` every 100ms

**apps/ws** - WebSocket Server (Bun runtime)
- Subscribes to Redis channel `ws:price:update`
- Broadcasts price updates to connected clients
- Runs on port 8080 by default

**apps/web** - React Frontend (Vite + TypeScript)
- UI built with shadcn/ui (Radix UI + TailwindCSS)
- React Query for state management and API calls
- Lightweight Charts for candlestick visualization
- WebSocket client for real-time price feed
- In-memory price store (`price-store.ts`) for O(1) lookups
- In-memory candlestick store (`candlestick-store.ts`) for chart data

**apps/mobile** - React Native/Expo App
- Cross-platform mobile app for iOS/Android
- Uses Expo Router for navigation
- React Query for data fetching
- Similar architecture to web app

### Packages

**packages/db** - Prisma ORM
- PostgreSQL schema for `User` and `ExistingTrade` models

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shashank-poola/tradex-distributed-system](https://github.com/shashank-poola/tradex-distributed-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
