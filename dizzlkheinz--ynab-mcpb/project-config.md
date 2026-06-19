---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Model Context Protocol (MCP) server for YNAB (You Need A Budget) integration, enabling AI assistants to interact with YNAB budgets, accounts, transactions, and categories. The codebase uses TypeScript with a modular, service-oriented architecture.

**Current Version:** 0.26.10

## Essential Commands

### Build & Development

```bash
npm run build              # Clean, compile TypeScript, and bundle
npm run build:no-lint      # Build without running linter
npm run build:prod         # Production build with verification
npm run dev                # TypeScript watch mode for development
npm run type-check         # Run TypeScript type checking without emitting files
```

### Testing

```bash
npm test                           # Run all unit tests + filter results
npm run test:unit                  # Unit tests only (fast, mocked dependencies)
npm run test:integration           # Integration tests (core only)
npm run test:integration:core      # Core integration tests
npm run test:integration:domain    # Domain-specific integration tests
npm run test:integration:full      # Full integration test suite (throttled)
npm run test:integration:budgets   # Budget-specific integration tests
npm run test:integration:accounts  # Account-specific integration tests
npm run test:integration:transactions  # Transaction-specific integration tests
npm run test:integration:categories    # Category-specific integration tests
npm run test:integration:payees    # Payee-specific integration tests
npm run test:integration:months    # Month-specific integration tests
npm run test:integration:delta     # Delta caching integration tests
npm run test:integration:reconciliation  # Reconciliation integration tests
npm run test:e2e                   # End-to-end tests (requires real YNAB token)
npm run test:performance           # Performance and load tests
npm run test:coverage              # Generate coverage report (requires 80% coverage)
npm run test:watch                 # Watch mode for test development
npm run test:comprehensive         # Run comprehensive test suite
npm run test:all                   # Run all tests (unit, integration, e2e, performance)
```

### Code Quality

```bash
npm run lint               # Run Biome linting
npm run lint:fix           # Auto-fix Biome lint/format issues
npm run format             # Format code with Biome
npm run format:check       # Check formatting without modifying files
```

### Packaging & Distribution

```bash
npm run package:mcpb        # Build production MCPB package for Claude Desktop
npm run generate:mcpb       # Generate MCPB file from built bundle
npm run bundle              # Bundle with esbuild (development)
npm run bundle:prod         # Bundle with minification (production)
npm run prepare             # Prepare package for publication (runs build:prod)
npm run prepublishOnly      # Pre-publish checks (runs tests + build)
```

## Architecture Overview

The architecture is modular and service-oriented:

### Core Server Components (`src/server/`)

- **YNABMCPServer.ts** - Main orchestration server, coordinates all services
- **toolRegistry.ts** - Centralized tool metadata, validation, execution, and progress notification support
- **completions.ts** - MCP completions manager for autocomplete suggestions (budget_id, account_id, category, payee)
- **cacheManager.ts** - Enhanced caching with LRU eviction, observability, and stale-while-revalidate
- **deltaCache.ts** - Delta request management with server knowledge tracking and merge operations
- **deltaCache.merge.ts** - Entity merging functions for delta responses (transactions, categories, accounts)
- **serverKnowledgeStore.ts** - Tracks last known server_knowledge values per cache key for delta requests
- **budgetResolver.ts** - Consistent budget ID resolution across all tools
- **errorHandler.ts** - Centralized error handling with dependency injection
- **config.ts** - Environment validation and server configuration
- **resources.ts** - MCP resource definitions and handlers (includes resource templates)
- **prompts.ts** - MCP prompt definitions and handlers
- **diagnostics.ts** - System diagnostics and health monitoring
- **securityMiddleware.ts** - Security validation and wrapper functions
- **responseFormatter.ts** - JSON response formatting (pretty-print)
- **markdownFormatter.ts** - Human-readable markdown output for all read tools (tables, detail views, pagination footers)
- **rateLimiter.ts** - Rate limiting for YNAB API compliance
- **requestLogger.ts** - Request/response logging middleware
- **cacheKeys.ts** - Centralized cache key generation utilities

### Tool Registration Pattern (2025-12)

- `ToolContext` (`src/types/toolRegistration.ts`) centralizes shared deps (ynabAPI, deltaFetcher/cache, knowledge store, default budget accessors, cache/diagnostic managers).
- Adapter helpers (`src/tools/adapters.ts`): `adapt`, `adaptNoInput`, `adaptWithDelta`, `adaptWrite`, and `createBudgetResolver` to inject default budget IDs; covered by unit tests in `src/tools/__tests__/adapters.test.ts`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dizzlkheinz/ynab-mcpb](https://github.com/dizzlkheinz/ynab-mcpb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
