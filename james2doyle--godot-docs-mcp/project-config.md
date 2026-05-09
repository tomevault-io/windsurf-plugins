---
trigger: always_on
description: This document provides coding guidelines and operational instructions for agents working on this codebase.
---

# Agent Guidelines for godot-docs-mcp

This document provides coding guidelines and operational instructions for agents working on this codebase.

## Project Overview

- **Type**: Cloudflare Workers MCP server (Model Context Protocol)
- **Purpose**: Provides AI agents with tools to search and retrieve Godot documentation
- **Runtime**: Cloudflare Workers (workerd) with Durable Objects
- **Language**: TypeScript (strict mode)
- **Testing**: Vitest with `@cloudflare/vitest-pool-workers`

## Architecture

```
src/
├── index.ts          # Main entry point, MyMCP agent class, fetch handler
├── utils.ts          # Search and docs retrieval logic
└── indexes/         # Search index JSON files (stable/, latest/, 4.6/, 4.5/, 4.4/, 4.3/)
tests/
└── *.test.ts        # Vitest tests with Cloudflare Workers pool
```

## MCP Server Configuration

The project uses Opencode MCP servers defined in `opencode.json`:

```json
{
  "mcp": {
    "cloudflare-docs": {
      "enabled": true,
      "type": "remote",
      "url": "https://docs.mcp.cloudflare.com/mcp"
    },
    "cloudflare-worker-bindings": {
      "enabled": false,
      "type": "remote",
      "url": "https://bindings.mcp.cloudflare.com/mcp"
    }
  }
}
```

- **cloudflare-docs**: Enabled - provides tools to search Cloudflare documentation
- **cloudflare-worker-bindings**: Disabled by default - provides tools to query Worker bindings

## Build, Lint, and Test Commands

```bash
# Install dependencies
npm install

# Development server (wrangler dev)
npm run dev
npm run dev:inspect    # With Node inspector for debugging

# Deploy to Cloudflare Workers
npm run deploy

# Type checking
npm run type-check     # tsc --noEmit

# Generate Cloudflare types
npm run cf-typegen    # wrangler types

# Formatting (Biome)
npm run format         # biome format --write
npm run lint:fix       # biome lint --fix

# Testing
npm test              # Run all tests (vitest)
npm test -- --run     # Run tests once (not watch mode)
npm test -- --run -t "test name"    # Run single test by name
npm test -- --run tests/search_docs.test.ts  # Run single test file
```

## Code Style

### Biome Configuration (biome.json)

The project uses Biome for formatting and linting:

- **Indent style**: Spaces (2 spaces)
- **Quote style**: Single quotes (`'`)
- **Line endings**: Auto (Git-based VCS enabled)
- **Linter**: Biome recommended rules enabled

Run `npm run format` before committing and fix lint errors with `npm run lint:fix`.

### TypeScript Conventions

- **Strict mode**: Enabled in `tsconfig.json`
- **Module resolution**: `bundler`
- **Target**: ES2021
- Use `type` imports for type-only imports:
  ```typescript
  import { type Options } from 'minisearch';
  ```
- Use inline types for simple objects when clear:
  ```typescript
  const output: { text: string; type: 'text' }[] = [];
  ```

### Naming Conventions

- **Classes**: PascalCase (`MyMCP`, `TurndownService`)
- **Functions/variables**: camelCase (`searchDocs`, `miniSearches`)
- **Types/interfaces**: PascalCase (`SearchIndexItem`, `Version`)
- **Constants**: camelCase or UPPER_SNAKE for config objects (`miniSearchOptions`)
- **Files**: kebab-case (`search_docs.test.ts`, `utils.ts`)

### Imports

- Use ES module syntax with `.js` extension for type imports from packages:
  ```typescript
  import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
  import { McpAgent } from 'agents/mcp';
  ```
- Organize imports: external packages first, then local imports
- Use Biome's auto-import organization (`npm run lint:fix` handles this)

### Error Handling

- MCP tools return a specific response format:
  ```typescript
  return {
    content: [
      {
        type: 'text',
        text: 'result text',
      },
    ],
    isError: true,  // optional, defaults to false
  };
  ```
- Use `isError: true` for error responses
- Log important events with `console.info()` or `console.error()`

## Testing Conventions

### Test Structure

Tests use Vitest with the Cloudflare Workers pool:

```typescript
import { describe, expect, it } from 'vitest';
import { searchDocs } from '../src/utils';

describe('feature name', () => {
  it('should do something', async () => {
    const results = await searchDocs('node3d');
    expect(results).toMatchObject({ /* expected shape */ });
  });
});
```

### Running Tests

- All tests: `npm test -- --run`
- Single test file: `npm test -- --run tests/search_docs.test.ts`
- Single test by name: `npm test -- --run -t "responds with results"`
- Tests run against the Cloudflare Workers runtime via `@cloudflare/vitest-pool-workers`

## Git Conventions

### Commit Messages

Use conventional commits (seen in git history):

- `feat: add new feature` - New functionality
- `fix: resolve bug` - Bug fixes
- `chore: update dependencies` - Maintenance, deps
- `docs: add documentation` - Documentation changes
- `refactor: reorganize code` - Code restructuring

### Branching

- Main branch: `main`
- Feature branches: `feat/description` or similar descriptive names

## Common Development Tasks

### Adding a New Tool

1. Add tool definition in `src/index.ts` using the MCP SDK:
   ```typescript
   this.server.tool(
     'tool_name',
     'Description of what it does',

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [james2doyle/godot-docs-mcp](https://github.com/james2doyle/godot-docs-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
