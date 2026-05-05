---
trigger: always_on
description: This document guides AI coding agents working on the gaia-agent codebase. Focus on architecture patterns, critical workflows, and project-specific conventions.
---

# Copilot Instructions for gaia-agent

This document guides AI coding agents working on the gaia-agent codebase. Focus on architecture patterns, critical workflows, and project-specific conventions.

## Architecture Overview

**Core Pattern: Factory + Strategy + Adapter**

- **Factory Pattern**: `createMemoryTools()`, `createSandboxTools()` - Abstract provider selection
- **Strategy Pattern**: `IMemoryProvider`, `ISandboxProvider` - Swappable runtime implementations
- **Adapter Pattern**: Unified tool interfaces despite different provider APIs (e.g., Mem0 vs AWS AgentCore)

**Tool Categories**: Memory, Sandbox, Search, Browser, Core (16+ tools total)

**Provider System**: Each tool category supports multiple swappable providers:
- Memory: `mem0` (Mem0 API) | `agentcore` (AWS Bedrock AgentCore Memory)
- Sandbox: `e2b` (E2B cloud sandbox) | `sandock` (Sandock API)
- Search: `tavily` (Q&A search) | `exa` (neural search with similarity/content APIs)

**Module Structure** (Exemplar: `src/tools/memory/`):
```
types.ts       # IMem0Provider, IAgentCoreProvider interfaces + IMemorySchemas
mem0.ts        # Mem0 provider implementation with schemas
agentcore.ts   # AWS AgentCore provider implementation
index.ts       # Factory functions: createMemoryTools(), createMemoryStoreTool(), etc.
README.md      # Pattern documentation + usage examples
```

## Critical Developer Workflows

**Build & Type Check**:
```bash
pnpm build       # TypeScript compilation (tsc) → dist/
pnpm typecheck   # Type checking without emit (tsc --noEmit)
```

**Code Quality** (Biome - all-in-one formatter/linter):
```bash
pnpm format      # Auto-format code (biome format --write)
pnpm lint        # Lint with auto-fix (biome lint --write)
pnpm check       # Format + lint + organize imports (biome check --write)
```

**Testing** (Vitest):
```bash
pnpm test              # Run all tests
pnpm test:watch        # Watch mode (re-runs on changes)
pnpm test:ui           # Interactive UI
pnpm test:coverage     # Coverage report
```

**Benchmarking** (GAIA official benchmark - modular architecture):
```bash
pnpm benchmark                 # Run validation set
pnpm benchmark:test            # Run test set
pnpm benchmark:level1          # Filter by difficulty level
pnpm benchmark:quick           # 5 tasks with verbose output
pnpm benchmark:random          # Random single task
pnpm benchmark:random --stream # Stream mode (real-time agent thinking)
```

## Adding New Providers

**Step 1**: Define interface in `types.ts`:
```typescript
// src/tools/memory/types.ts
export interface IMem0Provider {
  add(messages: { role: string; content: string }[]): Promise<{ results: { id: string }[] }>;
  search(query: string): Promise<{ results: Array<{ memory: string }> }>;
  // ... other methods
}
```

**Step 2**: Implement provider in separate file:
```typescript
// src/tools/memory/mem0.ts
import type { IMem0Provider, IMemorySchemas } from './types';

export function createMem0Provider(apiKey: string): IMem0Provider {
  return {
    async add(messages) { /* ... */ },
    async search(query) { /* ... */ },
  };
}

export const mem0Schemas: IMemorySchemas = {
  add: z.object({ messages: z.array(z.object({ ... })) }),
  search: z.object({ query: z.string() }),
  // ... other schemas
};
```

**Step 3**: Update factory in `index.ts`:
```typescript
// src/tools/memory/index.ts
export function createMemoryTools(
  provider: 'mem0' | 'agentcore',
  config: MemoryConfig
): Record<string, Tool<any, any>> {
  const impl = provider === 'mem0' 
    ? createMem0Provider(config.apiKey)
    : createAgentCoreProvider(config);
  const schemas = provider === 'mem0' ? mem0Schemas : agentcoreSchemas;

  return {
    memoryStore: createMemoryStoreTool(impl, schemas),
    memorySearch: createMemorySearchTool(impl, schemas),
    // ... other tools
  };
}
```

**Step 4**: Preserve backward compatibility (legacy exports):
```typescript
// For existing code using direct imports
export { mem0Search, mem0Store } from './mem0';
```

## Project-Specific Conventions

**Language Requirements**:
- **Use English only** for all code, comments, documentation, and commit messages
- **No other languages allowed** - No Chinese, Japanese, Korean, or any non-English characters in the codebase
- Variable names, function names, comments, JSDoc, and error messages must be in English
- Documentation files (README, ARCHITECTURE, etc.) must be in English
- Error messages and user-facing strings must be in English

**Type Safety**:
- All tools use Zod schemas for validation
- AI SDK Tool type casting: `inputSchema: schema as unknown as Tool["inputSchema"]`
- Strict TypeScript mode enabled (`strict: true`, `noImplicitAny: true`)

**ESM & Tree-Shaking**:
- Package uses ESM modules (`"type": "module"`)
- Granular exports in `package.json` for tree-shaking:
  ```json
  {
    ".": "./dist/index.js",
    "./tools/memory": "./dist/tools/memory/index.js",
    "./tools/sandbox": "./dist/tools/sandbox/index.js"
  }
  ```

**File Organization**:
- `src/` - Source TypeScript files (library code)
- `dist/` - Compiled JavaScript (git-ignored)
- `benchmark/` - Modular benchmark runner (excluded from build)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gaia-agent/gaia-agent](https://github.com/gaia-agent/gaia-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
