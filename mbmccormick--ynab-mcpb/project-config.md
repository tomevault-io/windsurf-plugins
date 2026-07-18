---
trigger: always_on
description: This document provides context and guidance for AI assistants working on the YNAB MCPB (Model Context Protocol Bundle) extension for Claude Desktop.
---

# CLAUDE.md - AI Assistant Guide for YNAB MCPB

This document provides context and guidance for AI assistants working on the YNAB MCPB (Model Context Protocol Bundle) extension for Claude Desktop.

## Project Overview

**Purpose**: A Claude Desktop extension that provides seamless integration with the You Need A Budget (YNAB) API, enabling budget management directly from Claude conversations.

**Type**: MCPB extension using Model Context Protocol (MCP)
**Platform**: Cross-platform (macOS, Linux, Windows)
**Language**: JavaScript (ES6 modules)
**API**: YNAB REST API v1

## Architecture

### High-Level Design

```
Claude Desktop
    ↓ (MCP over stdio)
server/index.js (MCP Server)
    ↓
server/ynab-client.js (HTTP Client)
    ↓ (HTTPS + Bearer Token)
YNAB REST API
```

### Key Design Patterns

1. **Lazy Initialization**: The YNAB client is initialized only when first tool is called, not in constructor. This allows the server to start successfully even without an API token configured.

2. **Response Formatting**: All large responses go through formatters in `server/response-formatter.js` to prevent 1MB tool result limits and reduce token usage.

3. **Modular Architecture**: Clear separation of concerns across files:
   - `index.js`: MCP protocol handling and routing
   - `ynab-client.js`: API communication
   - `tool-definitions.js`: Tool schemas
   - `response-formatter.js`: Data optimization
   - `utils.js`: Shared utilities

## File Structure

```
ynab-mcpb/
├── manifest.json              # MCPB extension manifest (critical: user_config section)
├── package.json               # Dependencies and scripts
├── README.md                  # User documentation
├── CLAUDE.md                  # This file - AI assistant guide
├── LICENSE                    # MIT License
├── icon.png                   # YNAB logo (234KB PNG)
├── .gitignore                # Git ignore rules
├── .mcp.json                  # MCP configuration (local dev only)
└── server/
    ├── index.js               # Main MCP server (9.5KB)
    ├── tool-definitions.js    # 24 MCP tool schemas (15KB)
    ├── ynab-client.js         # YNAB API client (8.4KB)
    ├── response-formatter.js  # Response optimization (7.3KB)
    ├── server-config.js       # Configuration constants (0.5KB)
    └── utils.js               # Utilities and helpers (2.6KB)
```

## Critical Technical Concepts

### 1. Milliunits

YNAB uses "milliunits" for all currency amounts:
- **1000 milliunits = $1.00**
- Example: $45.32 = 45,320 milliunits
- Positive = inflow (income)
- Negative = outflow (expenses)

**Always use `MilliunitConverter` utilities** from `utils.js`:
```javascript
import { MilliunitConverter } from './utils.js';

// Converting to milliunits
const milliunits = MilliunitConverter.toMilliunits(45.32); // 45320

// Formatting for display
const formatted = MilliunitConverter.formatCurrency(45320); // "$45.32"
```

### 2. User Configuration (API Token)

The YNAB API token is configured via `user_config` in `manifest.json`:

```json
"user_config": {
  "YNAB_API_TOKEN": {
    "type": "string",
    "sensitive": true,
    "required": true
  }
}
```

Claude Desktop automatically:
- Prompts user for token during installation
- Injects it as environment variable: `process.env.YNAB_API_TOKEN`
- Stores it securely (never in code or logs)

**Important**: Use `${user_config.YNAB_API_TOKEN}` syntax in manifest, not `${YNAB_API_TOKEN}`.

### 3. Response Formatters

**Critical for preventing errors**: Raw YNAB API responses can be massive (18MB+). Always use formatters for large responses.

Example:
```javascript
// BAD - Returns 18MB of data
case "get_budget":
  return await client.getBudget(args.budget_id);

// GOOD - Returns ~1KB summary
case "get_budget":
  const result = await client.getBudget(args.budget_id);
  return formatBudget(result);
```

**When to use formatters**:
- `get_budget` → `formatBudget()` (18MB → 1KB)
- `get_accounts` → `formatAccounts()` (20KB → 5KB)
- `get_categories` → `formatCategories()` (100KB → 10KB)
- `get_month` → `formatMonth()` (200KB → 20KB)
- `get_transactions` → `formatTransactions()` (limits to 50)
- `get_payees` → `formatPayees()` (limits to 100)
- `get_scheduled_transactions` → `formatScheduledTransactions()` (limits to 50)

### 4. Lazy Client Initialization

The YNAB client must not be initialized in constructor:

```javascript
// server/index.js
class YnabServer {
  constructor() {
    this.ynabClient = null; // Don't initialize here
  }

  getYnabClient() {
    if (!this.ynabClient) {
      const apiToken = process.env.YNAB_API_TOKEN;
      if (!apiToken) {
        throw new McpError(...); // Friendly error message
      }
      this.ynabClient = new YnabClient(apiToken);
    }
    return this.ynabClient;
  }
}
```

**Why**: Allows server to start successfully without token, enabling proper error handling when tools are actually called.

## Common Development Tasks

### Adding a New Tool

1. **Add tool definition** in `server/tool-definitions.js`:
```javascript
{
  name: "new_tool_name",
  description: "Clear description of what this tool does",
  inputSchema: {
    type: "object",
    properties: {
      budget_id: {
        type: "string",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mbmccormick/ynab-mcpb](https://github.com/mbmccormick/ynab-mcpb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
