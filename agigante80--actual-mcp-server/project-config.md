---
trigger: always_on
description: **Actual MCP Server** bridges AI assistants with [Actual Budget](https://actualbudget.org/) via the Model Context Protocol (MCP), providing **63 tools** for conversational financial management. Supports two transports: **HTTP** (LibreChat/LobeChat/multi-user) and **stdio** (Claude Desktop/Claude Code).
---

# Copilot Instructions for Actual MCP Server

## Project Overview

**Actual MCP Server** bridges AI assistants with [Actual Budget](https://actualbudget.org/) via the Model Context Protocol (MCP), providing **63 tools** for conversational financial management. Supports two transports: **HTTP** (LibreChat/LobeChat/multi-user) and **stdio** (Claude Desktop/Claude Code).

**Tech Stack**: TypeScript (NodeNext/ESM), Node.js 20+, `@actual-app/api` v26, `@modelcontextprotocol/sdk`, Express 5, Zod v4, Playwright

**Current Status**: Production-ready, 63 tools implemented

## Architecture Essentials

### Critical Pattern: `withActualApi` Wrapper

**Every Actual API operation MUST use `withActualApi()` wrapper** from `src/lib/actual-adapter.ts`:

```typescript
// ✅ CORRECT - ensures data persistence
await withActualApi(async () => {
  return await rawAddTransactions(data);
});

// ❌ WRONG - data won't persist (tombstone issue)
await rawAddTransactions(data);
```

**Why**: Actual Budget requires `api.shutdown()` after every operation to commit data. The `withActualApi` wrapper handles init/shutdown lifecycle automatically. Based on [s-stefanov/actual-mcp](https://github.com/s-stefanov/actual-mcp) pattern.

### Tool Structure

New tools should use `createTool()` from `src/lib/toolFactory.ts` — it wires up error handling, logging, and observability automatically:

```typescript
import { z } from 'zod';
import { createTool } from '../lib/toolFactory.js';
import { CommonSchemas } from '../lib/schemas/common.js';
import adapter from '../lib/actual-adapter.js';

export default createTool({
  name: 'actual_domain_action',      // naming: actual_{domain}_{action}
  description: '...',
  schema: z.object({
    account: CommonSchemas.accountId,
    amount: CommonSchemas.amountCents, // always in cents, integer
    date: CommonSchemas.date,          // YYYY-MM-DD
  }),
  handler: async (input) => {
    return await adapter.someMethod(input);
  },
});
```

Many existing tools use the legacy pattern (both work, but `createTool()` is preferred for new tools):

```typescript
import type { ToolDefinition } from '../../types/tool.d.js';
const InputSchema = z.object({ ... });
const tool: ToolDefinition = {
  name: 'actual_transactions_create',
  description: '...',
  inputSchema: InputSchema,
  call: async (args: unknown) => {
    const input = InputSchema.parse(args);
    return await adapter.addTransactions(input);
  },
};
export default tool;
```

**Key conventions**:
- Tool names: `actual_{domain}_{action}` (e.g., `actual_accounts_create`)
- Amounts: **always in cents** (integer), never dollars
- Dates: `YYYY-MM-DD` format (validated by `CommonSchemas.date`)
- UUIDs: Validated with `UUID_PATTERN` from `src/lib/constants.ts`
- Shared schemas: Use `CommonSchemas` from `src/lib/schemas/common.ts` for consistency

### Module Organization

```
src/
├── index.ts                    # Entry point, CLI parsing, server startup
├── actualConnection.ts         # Actual Budget connection lifecycle
├── actualToolsManager.ts       # Tool registry (63 tools in IMPLEMENTED_TOOLS array), dispatch, validation
├── auth/
│   ├── setup.ts                # createMcpAuth() factory (MCPAuth singleton, AUTH_PROVIDER=oidc)
│   └── budget-acl.ts           # Per-user budget ACL (email/sub/group principals, AUTH_BUDGET_ACL)
├── lib/
│   ├── actual-adapter.ts       # ⚠️ CRITICAL: withActualApi wrapper, retry logic
│   ├── ActualMCPConnection.ts  # MCP protocol implementation (EventEmitter-based)
│   ├── retry.ts                # Exponential backoff retry (3 attempts, 200ms base)
│   ├── constants.ts            # All configuration constants, patterns, limits
│   ├── schemas/common.ts       # Shared Zod schemas (accountId, amountCents, etc.)
│   └── loggerFactory.ts        # Module-scoped loggers (winston)
├── server/
│   └── httpServer.ts           # HTTP transport
└── tools/                      # 63 tool definitions (see actualToolsManager.ts)
```

## Development Workflow

### Build & Run Commands

```bash
npm run build                   # TypeScript compilation (required before running)
npm run dev -- --http --debug   # Development mode with HTTP transport + debug logs
npm run start                   # Production mode (requires build first)

# Testing
npm run test:adapter            # Adapter smoke tests (concurrency, retry logic)
npm run test:unit-js            # All unit tests (4 files)
node tests/unit/transactions_create.test.js          # Run a single unit test file
node tests/unit/schema_validation.test.js            # Schema validation tests only
npm run test:e2e                # Playwright E2E tests (initialize → tools/call → streaming)
npx playwright test --grep "initialize -> tools/list" # Single E2E test by name

# Tool Management
npm run verify-tools            # Verify all 63 tools are correctly registered
npm run check:coverage          # List @actual-app/api methods vs current tool coverage

# Debugging
npm run test:mcp-client         # Connect as MCP client and exercise tools (requires build)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agigante80/actual-mcp-server](https://github.com/agigante80/actual-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
