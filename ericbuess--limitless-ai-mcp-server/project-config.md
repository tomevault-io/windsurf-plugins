---
trigger: always_on
description: > 🤖 **Purpose**: This document provides essential information for Claude and other AI assistants to effectively work on this project. It includes project structure, development commands, implementation details, and troubleshooting guidance.
---

# Limitless AI MCP Server - Claude Development Guide

> 🤖 **Purpose**: This document provides essential information for Claude and other AI assistants to effectively work on this project. It includes project structure, development commands, implementation details, and troubleshooting guidance.

## Project Overview

This is a Model Context Protocol (MCP) server that enables AI assistants to interact with the Limitless AI API, specifically for accessing Pendant recordings (lifelogs). The server provides structured tools for searching, listing, and retrieving recording data.

## Project Structure

```
limitless-ai-mcp-server/
├── src/                    # Source code
│   ├── core/              # Core business logic
│   │   ├── limitless-client.ts    # API client implementation
│   │   └── cache.ts       # LRU cache with TTL support
│   ├── tools/             # MCP tool definitions
│   │   ├── definitions.ts # Tool metadata and descriptions
│   │   ├── handlers.ts    # Tool implementation handlers
│   │   └── schemas.ts     # Zod schemas for validation
│   ├── resources/         # MCP Resources feature
│   │   ├── handlers.ts    # Resource request handlers
│   │   └── manager.ts     # Resource management logic
│   ├── prompts/           # MCP Prompts feature
│   │   ├── handlers.ts    # Prompt request handlers
│   │   └── templates.ts   # Prompt template definitions
│   ├── sampling/          # MCP Sampling feature
│   │   ├── handlers.ts    # Sampling request handlers
│   │   └── templates.ts   # Sampling templates
│   ├── types/             # TypeScript type definitions
│   │   ├── limitless.ts   # API and domain types
│   │   ├── cache.ts       # Cache type definitions
│   │   └── mcp.ts         # MCP-specific types
│   ├── utils/             # Utility functions
│   │   ├── date.ts        # Date formatting/parsing
│   │   ├── format.ts      # Response formatting
│   │   ├── logger.ts      # Logging utility
│   │   └── retry.ts       # Retry logic with exponential backoff
│   └── index.ts           # Main server entry point
├── tests/                 # Test files
│   ├── core/
│   │   └── cache.test.ts  # Cache tests (20 tests)
│   ├── prompts/
│   │   └── handlers.test.ts # Prompt tests (8 tests)
│   ├── resources/
│   │   └── manager.test.ts # Resource tests (11 tests)
│   ├── sampling/
│   │   └── handlers.test.ts # Sampling tests (8 tests)
│   └── utils/
│       └── retry.test.ts  # Retry utility tests
├── examples/              # Usage examples
│   ├── basic-usage.ts     # Simple client usage
│   ├── using-tools.ts     # Demonstrate all 5 tools
│   ├── using-resources.ts # Show resource browsing
│   ├── using-prompts.ts   # Use each prompt template
│   ├── using-sampling.ts  # Content analysis demos
│   ├── advanced-search.ts # Complex search patterns
│   └── caching-strategies.ts # Performance optimization
├── docs/                  # Documentation
│   └── references/        # Reference documentation
│       ├── llms-full_model-context-protocol_20250601.md
│       └── limitless-api-docs_20250601.md
├── dist/                  # Compiled JavaScript output
├── package.json           # Project dependencies and scripts
├── tsconfig.json          # TypeScript configuration
├── jest.config.js         # Jest test configuration
├── LICENSE                # MIT License
└── README.md              # User documentation
```

## Key Commands

### Development Commands

```bash
# Install dependencies
npm install

# Build the project (TypeScript → JavaScript)
npm run build

# Run in development mode with hot reload
npm run dev

# Run tests
npm test

# Run tests in watch mode
npm run test:watch

# Generate test coverage report
npm run test:coverage

# Lint the code
npm run lint

# Format code with Prettier
npm run format

# Type check without building
npm run typecheck
```

### Git Commands

```bash
# Current branch: dev
git status
git add .
git commit -m "commit message"
git push origin dev

# Create pull request to main
gh pr create --title "PR title" --body "description"
```

## Environment Variables

Required:

- `LIMITLESS_API_KEY` - API key from limitless.ai/developers (required)

Optional:

- `LIMITLESS_BASE_URL` - API base URL (default: https://api.limitless.ai/v1)
- `LIMITLESS_TIMEOUT` - Request timeout in ms (default: 120000)
- `LOG_LEVEL` - Logging level: DEBUG, INFO, WARN, ERROR (default: INFO)

Cache Configuration:

- `CACHE_MAX_SIZE` - Maximum items in lifelog cache (default: 100)
- `CACHE_TTL` - Lifelog cache TTL in ms (default: 300000 / 5 minutes)
- `SEARCH_CACHE_MAX_SIZE` - Maximum items in search cache (default: 50)
- `SEARCH_CACHE_TTL` - Search cache TTL in ms (default: 180000 / 3 minutes)

## API Authentication

The Limitless API uses `X-API-Key` header authentication (NOT Bearer tokens):

```typescript
headers: {
  'X-API-Key': apiKey,
  'Content-Type': 'application/json'
}
```

## Available MCP Tools

1. **limitless_get_lifelog_by_id** - Get a specific recording by ID
2. **limitless_list_lifelogs_by_date** - List recordings for a specific date

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ericbuess/limitless-ai-mcp-server](https://github.com/ericbuess/limitless-ai-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
