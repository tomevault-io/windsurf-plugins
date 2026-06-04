---
trigger: always_on
description: Jupalyse is a web-based analytics and tracking tool for Jupiter DCAs (Dollar Cost Averaging) on Solana. It helps users monitor, analyze, and download data from their Jupiter recurring and trigger orders for tax reporting and personal record-keeping.
---

# Jupalyse - AI Assistant Guide

## Project Overview

Jupalyse is a web-based analytics and tracking tool for Jupiter DCAs (Dollar Cost Averaging) on Solana. It helps users monitor, analyze, and download data from their Jupiter recurring and trigger orders for tax reporting and personal record-keeping.

**Key Features:**

- View all Jupiter DCAs for any Solana address
- Display all trades in an interactive, feature-rich table
- Download CSV data suitable for tax preparation
- Optional USD price fetching for comprehensive reporting
- **Privacy-first**: Runs entirely locally, user data never sent to external servers

**Live Site:** https://jupalyse.vercel.app

## Tech Stack

### Frontend

- **React 18.3.1** - Core UI framework
- **React Router 6.27.0** - Client-side routing with data loaders
- **Vite 5.4.8** - Build tool and dev server with fast HMR
- **TypeScript 5.5.3** - Strict type checking enabled
- **Mantine 7.13.3** - Component library with dark mode support
- **Tabler Icons React 3.19.0** - Icon library

### State Management & Data Fetching

- **React Query (TanStack Query) 5.62.0** - Server state management with caching
- **React Query Persist Client** - localStorage persistence for cached queries
- **React Query DevTools** - Development debugging tools

### Blockchain Integration

- **@solana/web3.js 2.0.0-rc.1** - Solana blockchain interaction

### Utilities

- **jdenticon 3.3.0** - Visual identicons for order keys
- **js-big-decimal 2.1.0** - Precise decimal arithmetic for token amounts

### Development & Deployment

- **Vercel** - Serverless API routes and hosting
- **pnpm** - Package manager (required, not npm/yarn)
- **ESLint 9.11.1** - Code linting with React plugins
- **Prettier 3.3.3** - Code formatting

## Project Structure

```
/home/sol/projects/Jupalyse/
├── src/                          # Frontend source code
│   ├── main.tsx                  # React app entry point with routing
│   ├── types.ts                  # Core TypeScript type definitions
│   ├── query-client.ts           # React Query configuration with persistence
│   ├── jupiter-api.ts            # Jupiter API client functions
│   ├── mint-data.ts              # Token metadata fetching
│   ├── token-prices.ts           # USD token price management (243 lines)
│   ├── number-display.ts         # Number formatting utilities
│   └── routes/                   # Page components
│       ├── root.tsx              # Root layout wrapper
│       ├── home.tsx              # Landing page with address input
│       ├── orders.tsx            # Order selection page
│       ├── trades.tsx            # Main trades table view (1,287 lines)
│       ├── trades-csv.ts         # CSV export logic
│       └── fetch-usd-prices.tsx  # USD price fetching action
├── api/                          # Vercel serverless API routes
│   ├── recurring-orders.ts       # Proxy for Jupiter recurring orders API
│   ├── trigger-orders.ts         # Proxy for Jupiter trigger orders API
│   └── token-search.ts           # Token search endpoint
├── public/                       # Static assets
├── dist/                         # Build output directory
├── vite.config.js                # Vite configuration with API proxy
├── tsconfig.json                 # TypeScript strict mode config
├── eslint.config.js              # ESLint flat config
├── postcss.config.cjs            # PostCSS & Mantine styling
├── vercel.json                   # Vercel deployment config
├── package.json                  # Dependencies and scripts
├── README.md                     # User-facing documentation
└── .env.copy                     # Environment variable template
```

## Development Setup

### Prerequisites

- **Node.js** (recent version)
- **pnpm** (REQUIRED - do not use npm or yarn)
- **Jupiter API key** from https://portal.jup.ag/api-keys

### Initial Setup

1. **Install dependencies:**

   ```bash
   pnpm install
   ```

2. **Configure environment:**

   ```bash
   cp .env.copy .env
   # Edit .env and set JUPITER_API_KEY
   ```

3. **Run local development (requires 2 terminals):**

   Terminal 1 - API routes:

   ```bash
   pnpm dev:api
   ```

   Terminal 2 - Frontend:

   ```bash
   pnpm dev
   ```

### Available Scripts

| Command         | Purpose                                                        |
| --------------- | -------------------------------------------------------------- |
| `pnpm dev`      | Start Vite dev server with hot module replacement              |
| `pnpm dev:api`  | Start Vercel local development for API routes (localhost:3000) |
| `pnpm build`    | TypeScript compilation + Vite production build                 |
| `pnpm lint`     | Check code with ESLint and Prettier                            |
| `pnpm lint:fix` | **Auto-fix linting and formatting issues**                     |
| `pnpm preview`  | Preview production build locally                               |

## Code Quality Guidelines

### After Making Changes

**ALWAYS run before committing:**

```bash
pnpm lint:fix
```

This auto-fixes:

- ESLint violations
- Prettier formatting issues

### TypeScript


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mcintyre94/Jupalyse](https://github.com/mcintyre94/Jupalyse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
