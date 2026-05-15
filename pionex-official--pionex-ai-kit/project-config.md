---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pionex AI Kit is a trading toolkit that provides AI agents with programmatic access to Pionex exchange via MCP (Model Context Protocol). The repository is a pnpm monorepo publishing two npm packages:

- **`@pionex/pionex-ai-kit`** — CLI for onboarding (writes `~/.pionex/config.toml`) and MCP client setup
- **`@pionex/pionex-trade-mcp`** — MCP server exposing Pionex trading tools to AI clients (Cursor, Claude Desktop, Windsurf, VS Code, etc.)
- **`@pionex-ai/core`** (private) — Shared utilities bundled into both public packages

## Repository Structure

```
packages/
  cli/     → @pionex/pionex-ai-kit (CLI + trade commands)
  mcp/     → @pionex/pionex-trade-mcp (MCP server)
  core/    → @pionex-ai/core (shared utilities)
    src/
      client/         → PionexRestClient (API wrapper)
      config/         → TOML config reading/writing
      tools/          → Tool definitions by module:
        market.ts     → Public market data (no auth)
        account.ts    → Account balance
        orders.ts     → Spot orders
        bot.ts        → Futures Grid Bot
      schemas/        → JSON schemas (futures grid validation)
      setup.ts        → MCP client config writers
```

## Build & Development

```bash
# Install dependencies
npm install

# Build all packages (core → cli → mcp)
npm run build

# Clean build artifacts
npm run clean

# Run built CLI
node packages/cli/dist/index.js help
node packages/cli/dist/index.js onboard

# Run built MCP server
node packages/mcp/dist/index.js --help
```

**Important:** Build order matters — `@pionex-ai/core` must be built before `cli` and `mcp` since they depend on it. The root `npm run build` handles this automatically.

## Architecture

### Three-Layer Design

1. **Core (`@pionex-ai/core`)** — Business logic, API client, tool definitions
   - `PionexRestClient`: REST API wrapper with signature authentication
   - Tool system: module-based tool registry (market, account, orders, bot)
   - Config: reads `~/.pionex/config.toml` for credentials

2. **CLI (`@pionex/pionex-ai-kit`)** — End-user commands
   - `onboard`: Interactive wizard to write `~/.pionex/config.toml`
   - `setup`: Writes MCP client config files (Cursor, Claude Desktop, etc.)
   - Direct trading commands (e.g., `pionex-trade-cli market depth BTC_USDT`)

3. **MCP Server (`@pionex/pionex-trade-mcp`)** — Protocol adapter
   - Implements MCP protocol (tools, capabilities)
   - Reads credentials from `~/.pionex/config.toml` or env vars
   - Maps MCP tool calls → core tool handlers → Pionex API

### Module System

Tools are organized into modules that can be enabled/disabled:

| Module | Tools | Auth Required |
|--------|-------|---------------|
| `market` | depth, trades, symbol_info, tickers, book_tickers, klines | No |
| `account` | get_balance | Yes |
| `orders` | new_order, get_order, cancel_order, get_fills, etc. | Yes |
| `bot` | futures_grid_create, adjust_params, reduce, cancel | Yes |

The MCP server's `system_get_capabilities` tool returns module availability (enabled/disabled/requires_auth) for agent planning.

### Credential Flow

Priority when MCP server starts:
1. Environment variables: `PIONEX_API_KEY`, `PIONEX_API_SECRET`, `PIONEX_BASE_URL`
2. `~/.pionex/config.toml` (fallback if env vars missing)

**Never write API keys into MCP client config files.** The setup command only writes the server start command (e.g., `npx @pionex/pionex-trade-mcp`).

## Key Implementation Patterns

### Tool Definition

Tools are defined in `packages/core/src/tools/*.ts` using the `ToolSpec` interface:

```typescript
{
  name: "pionex_market_get_depth",
  module: "market",
  isWrite: false,  // read-only flag
  description: "Get order book depth...",
  inputSchema: { type: "object", ... },
  async handler(args, { client, config }) {
    return (await client.publicGet("/api/v1/market/depth", { symbol })).data;
  }
}
```

The MCP server converts `ToolSpec` → MCP `Tool` format via `toMcpTool()` in `packages/core/src/tools/types.ts`.

### REST Client

`PionexRestClient` in `packages/core/src/client/rest-client.ts`:
- Public endpoints: `publicGet(path, query)`
- Authenticated: `privatePost(path, params)` (adds signature + timestamp)
- Error handling: `PionexApiError` for API failures

### Futures Grid Bot

The `bot` module (`packages/core/src/tools/bot.ts`) has complex order data validation via JSON schema in `packages/core/src/schemas/futures-grid-create.ts`. When adding/modifying bot tools:
- Update schema in `packages/core/src/schemas/`
- Validate input with `parseAndValidateCreateFuturesGridBuOrderData()`
- Export schema constants (e.g., `CREATE_FUTURES_GRID_ORDER_DATA_KEYS`) for CLI help text

## MCP Client Setup

The `setup` command (`packages/core/src/setup.ts`) writes config files for supported clients:

| Client | Config Path |
|--------|-------------|
| `cursor` | `~/.cursor/mcp.json` |
| `claude-desktop` | `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) |
| `claude-code` | Runs `claude mcp add` (no file written) |
| `windsurf` | `~/.codeium/windsurf/mcp_config.json` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pionex-official/pionex-ai-kit](https://github.com/pionex-official/pionex-ai-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
