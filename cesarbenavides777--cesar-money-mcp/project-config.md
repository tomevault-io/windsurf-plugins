---
trigger: always_on
description: TypeScript MCP server for Monarch Money personal finance data. Provides 20+ tools, MCP resources, and canned prompts for AI assistants to query accounts, transactions, budgets, cash flow, net worth, and more. Includes an analysis engine for spending breakdowns, anomaly detection, cash flow forecasting, subscription tracking, trend detection, and financial health scoring.
---

# Monarch Money MCP Server

## Project Overview

TypeScript MCP server for Monarch Money personal finance data. Provides 20+ tools, MCP resources, and canned prompts for AI assistants to query accounts, transactions, budgets, cash flow, net worth, and more. Includes an analysis engine for spending breakdowns, anomaly detection, cash flow forecasting, subscription tracking, trend detection, and financial health scoring.

Supports two transports:
- **stdio** -- for Claude Desktop and Claude Code
- **HTTP** (Streamable HTTP via Hono) -- for Claude Custom Connectors and remote clients, with full OAuth 2.1 + PKCE

## Tech Stack

- **Runtime**: Bun (>= 1.0)
- **HTTP Framework**: Hono
- **MCP**: `@modelcontextprotocol/sdk` (^1.12.0)
- **Finance API**: `monarchmoney` npm package (^1.1.3)
- **Auth**: OAuth 2.1 with PKCE, SQLite token storage via `bun:sqlite`
- **Validation**: Zod
- **TypeScript**: ^5.7, target ES2022, module ESNext, bundler resolution
- **CI/CD**: GitHub Actions → Fly.io

## Quick Navigation

| What you're looking for | Where to go |
|---|---|
| Entry point | `src/index.ts` |
| Environment config | `src/config.ts` |
| Monarch Money API client | `src/monarch/client.ts` |
| MCP server + tool registration | `src/mcp/server.ts` |
| HTTP transport bridge | `src/mcp/transport.ts` |
| Data tools | `src/tools/accounts.ts`, `transactions.ts`, `budgets.ts`, `cashflow.ts`, `recurring.ts`, `categories.ts`, `institutions.ts`, `insights.ts` |
| Analysis tools | `src/tools/analysis.ts` (wires analysis functions as MCP tools) |
| Analysis engine (pure functions) | `src/analysis/spending.ts`, `anomalies.ts`, `forecasting.ts`, `subscriptions.ts`, `trends.ts`, `health.ts` |
| MCP resources | `src/tools/resources.ts` |
| MCP prompts | `src/tools/prompts.ts` |
| OAuth 2.1 flow | `src/oauth/routes.ts`, `store.ts`, `provider.ts` |
| Middleware | `src/middleware/auth.ts`, `rate-limit.ts`, `audit.ts` |
| Tests | `src/**/*.test.ts` (colocated with source) |
| CI pipeline | `.github/workflows/ci.yml` |
| Deploy pipeline | `.github/workflows/deploy.yml` |
| Fly.io config | `fly.toml`, `Dockerfile` |
| monarchmoney SDK types | `node_modules/monarchmoney/dist/types/` |
| MCP SDK types | `node_modules/@modelcontextprotocol/sdk/dist/esm/server/mcp.d.ts` |

## Project Structure

```
src/
├── index.ts                  # Entry point: picks stdio or HTTP mode, bootstraps the app
├── config.ts                 # Loads env vars + CLI flags into a typed Config object
├── monarch/
│   └── client.ts             # Singleton MonarchClient with login, session caching, retry, and response caching
├── mcp/
│   ├── server.ts             # Creates McpServer and registers all tools, resources, and prompts
│   └── transport.ts          # HttpTransport: bridges Hono HTTP requests to MCP's Transport interface
├── tools/
│   ├── index.ts              # Barrel re-exports for all tool registration functions
│   ├── accounts.ts           # get_accounts, get_account_history
│   ├── transactions.ts       # get_transactions, search_transactions
│   ├── budgets.ts            # get_budgets
│   ├── cashflow.ts           # get_cashflow, get_cashflow_summary
│   ├── recurring.ts          # get_recurring_transactions
│   ├── categories.ts         # get_categories, get_category_groups
│   ├── institutions.ts       # get_institutions
│   ├── insights.ts           # get_net_worth, get_net_worth_history
│   ├── analysis.ts           # Wrappers that call analysis/ functions and expose them as MCP tools
│   ├── resources.ts          # MCP resource definitions (finance:// URIs)
│   └── prompts.ts            # MCP prompt templates (monthly-review, budget-check, etc.)
├── analysis/
│   ├── index.ts              # Barrel exports for all analysis functions and types
│   ├── spending.ts           # analyzeSpending — category breakdowns, top merchants, daily averages
│   ├── anomalies.ts          # detectAnomalies — large purchases, duplicates, unfamiliar merchants
│   ├── forecasting.ts        # forecastCashflow — 30/60/90-day balance projections
│   ├── subscriptions.ts      # analyzeSubscriptions — recurring payment analysis, price changes
│   ├── trends.ts             # detectTrends — multi-month category spending direction
│   └── health.ts             # calculateHealthScore — composite 0-100 financial health score
├── oauth/
│   ├── routes.ts             # Hono routes: .well-known, /oauth/register, /oauth/authorize, /oauth/token
│   ├── provider.ts           # validateMonarchCredentials — login-to-verify (no storage)
│   └── store.ts              # SQLite-backed store for auth codes, tokens, and dynamic clients
└── middleware/
    ├── auth.ts               # bearerAuth() — validates access tokens from Authorization header
    ├── rate-limit.ts         # rateLimit() — per-IP sliding window, returns 429 when exceeded
    └── audit.ts              # auditLog() — structured JSON log line per request with timing
```

## Key Patterns

### Adding a New Data Tool

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CesarBenavides777) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
