---
trigger: always_on
description: This is a Model Context Protocol (MCP) server that provides Orderly Network documentation and SDK patterns to AI assistants. It enables developers to query documentation, get code examples, and receive guidance on building trading UIs.
---

# AGENTS.md - Orderly MCP Server

## Project Overview

This is a Model Context Protocol (MCP) server that provides Orderly Network documentation and SDK patterns to AI assistants. It enables developers to query documentation, get code examples, and receive guidance on building trading UIs.

**Key Technologies:**

- TypeScript (ES modules)
- MCP SDK for tool/resource definitions
- Vitest for testing
- Yarn for package management
- ESLint + Prettier for code quality

## Architecture

**Entry Points:**

- `src/index.ts` - Stdio transport (for local MCP clients like Claude Desktop)
- `src/http-server.ts` - HTTP transport (for hosted deployments, Docker)
- `src/server.ts` - Shared MCP server logic (tools, resources, handlers)

**Transports:**

- **Stdio**: Default for local AI assistants, communicates via stdin/stdout
- **HTTP**: Stateless Streamable HTTP transport for remote access, runs on port 3000
- **Docker**: Runs HTTP mode by default with health checks

**Tools** (`src/tools/*.ts`):

- `searchDocs.ts` - Search documentation chunks
- `sdkPatterns.ts` - Get SDK hook patterns
- `contracts.ts` - Contract address lookup
- `workflows.ts` - Workflow explanations
- `apiInfo.ts` - API documentation
- `indexerApi.ts` - Indexer API documentation
- `componentGuides.ts` - Component building guides
- `orderlyOneApi.ts` - Orderly One API documentation

**Data** (`src/data/*.json`):

- Static JSON files with documentation, patterns, addresses
- Imported with `with { type: "json" }` syntax
- Read at runtime by tools

**Resources** (`src/resources/index.ts`):

- Handles URI-based resource requests
- Returns markdown or JSON content

## Common Tasks

### Build Project

```bash
yarn build
```

### Run Tests

```bash
yarn test:run          # Run all tests once
yarn test              # Watch mode
yarn test:coverage     # With coverage report
```

### Code Quality

```bash
yarn lint              # Check for issues
yarn lint:fix          # Fix auto-fixable issues
yarn format            # Format all files
yarn format:check      # Check formatting
yarn typecheck         # TypeScript check
```

### Development

```bash
yarn dev               # Watch mode build
yarn start             # Run built server (stdio mode)
yarn start:http        # Run HTTP server (port 3000)
```

### Docker

```bash
# Build Docker image
docker build -t orderly-mcp .

# Run container
docker run -p 3000:3000 orderly-mcp
```

## Project Structure

```
src/
├── index.ts                    # Stdio transport (for local MCP clients)
├── http-server.ts              # HTTP transport (for hosted deployments)
├── server.ts                   # Shared MCP server logic (tools, resources, handlers)
├── tools/                      # Tool implementations
│   ├── searchDocs.ts          # Doc search
│   ├── sdkPatterns.ts         # SDK patterns
│   ├── contracts.ts           # Contract lookup
│   ├── workflows.ts           # Workflows
│   ├── apiInfo.ts             # API info
│   ├── indexerApi.ts          # Indexer API info
│   ├── componentGuides.ts     # Component guides
│   └── orderlyOneApi.ts       # Orderly One API documentation
├── resources/
│   └── index.ts               # Resource handlers
├── data/                       # Static data
│   ├── documentation.json     # Searchable docs
│   ├── sdk-patterns.json      # SDK patterns
│   ├── contracts.json         # Contract addresses
│   ├── workflows.json         # Workflows
│   ├── api.json               # API docs
│   ├── indexer-api.json       # Indexer API docs
│   ├── orderly-one-api.json   # Orderly One API documentation
│   ├── component-guides.json   # Component guides
│   └── resources/
│       └── overview.md
└── __tests__/                  # Test files
    ├── contracts.test.ts
    └── searchDocs.test.ts
```

## Updating Documentation

The documentation is auto-generated using NEAR AI Cloud. All data files in `src/data/` are created by scripts, not manually edited.

### Data Generation Workflow

**Prerequisites:**

1. NEAR AI API key in `.env` file: `NEAR_AI_API_KEY=your_key`
2. Get API key at: https://cloud.near.ai/api-keys

**Option A: Complete Regeneration (Recommended)**

Generate everything from scratch using both Telegram chats and official docs:

```bash
# 1. Download latest official docs
curl -o llms-full.txt https://orderly.network/docs/llms-full.txt

# 2. Split Telegram export (if you have one)
node scripts/split_telegram_chats.js

# 3. Analyze Telegram chats → tg_analysis.json
node scripts/analyze_chat_openai.js

# 4. Analyze docs → docs_analysis.json
node scripts/analyze_llms_full.js

# 5. Generate all data files at once
node scripts/generate_mcp_data.js

# 6. Build and test
yarn build && yarn test:run
```

**Option B: Update Only Documentation**

If you just want to refresh from official docs without Telegram data:

```bash
# 1. Download latest docs
curl -o llms-full.txt https://orderly.network/docs/llms-full.txt

# 2. Analyze docs only
node scripts/analyze_llms_full.js

# 3. Generate (will use existing tg_analysis.json if present)
node scripts/generate_mcp_data.js

# 4. Build
yarn build
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OrderlyNetwork/orderly-mcp](https://github.com/OrderlyNetwork/orderly-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
