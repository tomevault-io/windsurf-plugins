---
trigger: always_on
description: MCPAdvisor is a TypeScript-based CLI tool for discovering and recommending MCP (Model Context Protocol) servers. It helps users find the right MCP server for their needs and provides installation guidance.
---

# Claude Instructions for MCPAdvisor

## Project Overview
MCPAdvisor is a TypeScript-based CLI tool for discovering and recommending MCP (Model Context Protocol) servers. It helps users find the right MCP server for their needs and provides installation guidance.

## Development Commands

### Build & Test
- `pnpm run build` - Compile TypeScript and make executable
- `pnpm run test` - Run tests with Vitest
- `pnpm run test:watch` - Run tests in watch mode
- `pnpm run test:coverage` - Run tests with coverage report
- `pnpm run test:jest` - Run Jest tests (alternative test runner)
- `pnpm run test:e2e` - Run end-to-end tests with Playwright
- `pnpm run test:meilisearch:e2e` - Smart Meilisearch E2E testing (auto-starts services)

### Code Quality
- `pnpm run lint` - Run ESLint
- `pnpm run lint:fix` - Run ESLint with auto-fix
- `pnpm run format` - Format code with Prettier
- `pnpm run format:check` - Check code formatting
- `pnpm run check` - Run both lint and format check

### Dependencies
- `pnpm run deps:update` - Update all dependencies to latest
- `pnpm run deps:check` - Check for outdated dependencies
- `pnpm run deps:clean` - Clean and reinstall dependencies

## Architecture

### Core Services
- **Search Service** (`src/services/searchService.ts`) - Main search orchestration
- **Vector Engines** (`src/services/database/`) - Vector database implementations
  - Memory-based vector engine
  - Meilisearch integration
  - OceanBase integration
  - Nacos integration
- **Installation Service** (`src/services/installation/`) - Installation guide generation
- **Server Service** (`src/services/server/`) - MCP server management

### Search Providers
- **Offline Search** - Memory-based search with local data
- **Meilisearch** - Full-text search with vector capabilities
- **Nacos** - Service discovery integration
- **Compass Search** - External search provider

### Key Directories
- `src/services/` - Core business logic
- `src/types/` - TypeScript type definitions
- `src/utils/` - Utility functions
- `src/tests/` - Test files (Vitest and Jest)
- `docs/` - Documentation files
- `config/` - Configuration files

## Code Style & Best Practices

### Naming Conventions
- **Classes**: PascalCase (e.g., `OfflineDataLoader`)
- **Functions/Methods**: camelCase (e.g., `loadFallbackData`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `DEFAULT_FALLBACK_DATA_PATH`)
- **Variables**: camelCase (e.g., `serverResponses`)
- **Interfaces**: PascalCase with `I` prefix (e.g., `IVectorSearchEngine`)
- **Files**: kebab-case (e.g., `offline-data-loader.ts`)

### Code Formatting
- Use 2 spaces for indentation
- Keep lines under 80 characters when possible
- Use template strings `` `${variable}` `` instead of concatenation
- Use single quotes `'` for strings, double quotes `"` in JSX
- Always use semicolons

### TypeScript Best Practices
- Always define types for function parameters and return values
- Avoid `any` type; use `unknown` or specific types
- Prefer interfaces over type aliases for objects
- Use union types `string | null` instead of optional `string?`
- Use enums for finite value sets
- Use type guards for type narrowing
- Use named exports instead of default exports

```typescript
// Good practice
export interface SearchOptions {
  minSimilarity?: number;
  limit: number;
}

export function search<T>(query: string, options: SearchOptions): Promise<T[]> {
  // implementation
}

// Avoid
export default function search(query: any, options?: any): any {
  // implementation
}
```

### Functional Programming Principles
- Write pure functions without side effects
- Avoid mutating parameters; return new objects
- Use function composition for complex functionality
- Follow single responsibility principle
- Use higher-order functions (`map`, `filter`, `reduce`)
- Avoid deep nesting; use function composition or Promise chains

```typescript
// Good practice
function normalizeVector(vector: number[]): number[] {
  const magnitude = calculateMagnitude(vector);
  return vector.map(value => value / magnitude);
}

// Avoid
function processVector(vector: number[]): number[] {
  // doing multiple things: calculate, normalize, filter, etc.
}
```

### Path Handling
- Use `pathUtils.ts` for all path-related operations
- Ensure compatibility across development, test, and production environments
- Handle `import.meta.url` compatibility properly
- Avoid hardcoded absolute paths; use relative paths
- Use `path.join()` and `path.resolve()` for path separators
- Provide fallback path mechanisms

```typescript
// Good practice
import { getMcpServerListPath } from '../utils/pathUtils.js';
const dataPath = getMcpServerListPath(import.meta.url);

// Avoid
const dataPath = path.resolve(__dirname, '../../../../data/file.json');
```

### Error Handling
- Use specific error types instead of generic `Error`
- Provide useful error messages with context
- Properly propagate errors; don't swallow them
- Use `try/catch` or Promise `.catch()` for async errors
- Log error details for debugging
- Provide graceful fallbacks when possible
- Scripts must return proper exit codes (0 for success, 1+ for failure)

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [istarwyh/mcpadvisor](https://github.com/istarwyh/mcpadvisor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
