---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`@gleanwork/mcp-server-tester` is a Playwright-based testing and evaluation framework for Model Context Protocol (MCP) servers. It provides Playwright fixtures for automated testing and data-driven eval datasets with optional LLM-as-a-judge scoring.

## Common Commands

```bash
# Build (includes UI reporter build)
npm run build

# Unit tests (Vitest)
npm test                    # Run all unit tests
npm run test:watch          # Watch mode
npm test -- src/mcp/clientFactory.test.ts  # Run single test file
npm test -- -t "creates client"            # Run tests matching pattern

# Integration tests (Playwright)
npm run test:playwright

# Code quality
npm run typecheck           # TypeScript validation
npm run lint                # ESLint
npm run lint:fix            # Auto-fix lint issues
npm run format              # Prettier formatting
npm run format:check        # Check formatting
```

## Architecture

### Core Modules (`src/`)

- **`config/`** - `MCPConfig` types and Zod validation for stdio/HTTP transports
- **`mcp/`** - Client factory (`createMCPClientForConfig`), fixtures (`MCPFixtureApi`), and response normalization
- **`auth/`** - OAuth 2.1 with PKCE (`PlaywrightOAuthClientProvider`) and static token utilities
- **`assertions/`** - Unified assertion architecture (see below)
- **`evals/`** - Dataset types, loader, and runner (uses validators internally)
- **`judge/`** - LLM-as-a-judge via Claude Agent SDK
- **`spec/`** - MCP protocol conformance checks
- **`reporters/`** - Custom Playwright reporter with React-based UI
- **`cli/`** - `mcp-server-tester init` and `mcp-server-tester generate` commands

### Assertions Module (`src/assertions/`)

The assertion architecture provides a single API for both inline tests and data-driven evals:

- **`validators/`** - Pure validation functions: `validateText`, `validateSchema`, `validatePattern`, `validateError`, `validateSize`, `validateResponse`, `validateToolCalls`, `validateToolCallCount`
- **`matchers/`** - Playwright custom matchers (see table below)

```typescript
// Inline test usage
import { expect } from '@gleanwork/mcp-server-tester';

test('weather tool', async ({ mcp }) => {
  const result = await mcp.callTool('get_weather', { city: 'London' });
  expect(result).toContainToolText('temperature');
  expect(result).toMatchToolSchema(WeatherSchema);
  expect(result).not.toBeToolError();
});

// Programmatic validation
import { validateText } from '@gleanwork/mcp-server-tester';

const result = validateText(response, ['temperature']);
if (!result.pass) console.log(result.message);
```

### Available Matchers

| Matcher                                  | Purpose                                       |
| ---------------------------------------- | --------------------------------------------- |
| `toMatchToolResponse(expected)`          | Exact response match (deep equal)             |
| `toContainToolText(text)`                | Response contains text substring(s)           |
| `toMatchToolPattern(pattern)`            | Response matches regex pattern(s)             |
| `toMatchToolSchema(schema)`              | Response validates against Zod schema         |
| `toMatchToolSnapshot(name, sanitizers?)` | Response matches saved snapshot               |
| `toBeToolError(expected?)`               | Response is (or is not) an error              |
| `toPassToolJudge(rubric, options?)`      | Response passes LLM-as-judge evaluation       |
| `toHaveToolResponseSize(options)`        | Response size is within bounds                |
| `toSatisfyToolPredicate(fn, desc?)`      | Response satisfies custom predicate           |
| `toHaveToolCalls(expectation)`           | LLM called the expected tools (mcp_host mode) |
| `toHaveToolCallCount(options)`           | LLM made N tool calls (mcp_host mode)         |

### Playwright Fixtures (`src/fixtures/mcp.ts`)

The main test fixture provides:

- `mcpClient: Client` - Raw MCP SDK client
- `mcp: MCPFixtureApi` - High-level test API with `listTools()`, `callTool()`, etc.

Configuration is read from `project.use.mcpConfig` in playwright.config.ts.

### Exports

Public API is defined in `src/index.ts`. The package has multiple export paths:

- `.` - Main library exports
- `./fixtures/mcp` - Playwright test fixtures
- `./fixtures/mcpAuth` - Auth-specific fixtures for OAuth/token auth
- `./reporters/mcpReporter` - Custom reporter

### Multi-Iteration Accuracy

Eval cases can be run multiple times to compute accuracy (win rate):

```json
{
  "id": "search-trigger",
  "mode": "mcp_host",
  "scenario": "Find recent docs about planning",
  "mcpHostConfig": { "provider": "anthropic" },
  "iterations": 5,
  "accuracyThreshold": 0.8,
  "expect": {
    "toolsTriggered": {
      "calls": [{ "name": "search", "required": true }]
    }
  }
}
```

- `iterations`: Run case N times (default: 1). When > 1, result has `assertionPassRate` (0-1) and `iterationResults[]`
- `accuracyThreshold`: Minimum accuracy to pass (default: 1.0)

### Concurrency

Run multiple eval cases in parallel:

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gleanwork/mcp-server-tester](https://github.com/gleanwork/mcp-server-tester) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
