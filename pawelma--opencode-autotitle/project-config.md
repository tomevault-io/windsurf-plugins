---
trigger: always_on
description: This document provides guidelines for AI coding agents working on the opencode-autotitle project.
---

# AGENTS.md - Coding Agent Guidelines

This document provides guidelines for AI coding agents working on the opencode-autotitle project.

## Project Overview

OpenCode plugin that automatically generates AI-powered session titles based on conversation context. Single-file TypeScript implementation (~480 lines) using the OpenCode Plugin SDK.

**Tech Stack:** TypeScript, Node.js >= 18, Bun >= 1.0, ESM modules

## Build/Lint/Test Commands

### Build Commands

```bash
# Build TypeScript to JavaScript
npm run build        # or: bun run build

# Watch mode for development
npm run dev          # or: bun run dev

# Type check without emitting files
npm run typecheck    # or: bun run typecheck
```

### Install Dependencies

```bash
bun install          # Preferred
npm install          # Alternative
```

### Test Commands

```bash
# Run tests once
npm test             # or: bun run test

# Run tests in watch mode
npm run test:watch   # or: bun run test:watch

# Run tests with coverage
npm run test:coverage
```

### Test Structure

Tests are located in `src/index.test.ts` using Vitest. The test suite covers:

- **Title detection**: `isTimestampTitle`, `hasPluginEmoji`, `shouldModifyTitle`
- **Text processing**: `sanitizeTitle`, `extractKeywords`, `inferIntent`, `generateFallbackTitle`
- **Model selection**: `findCheapestFromModels`, `CHEAP_MODEL_PATTERNS`
- **Configuration**: `loadConfig` (environment variable parsing)
- **Event extraction**: `extractSessionId`, `extractMessageContent`

When adding new functionality:
1. Export the function from `src/index.ts` if it's a pure function
2. Add corresponding tests in `src/index.test.ts`
3. Run `npm test` to verify all tests pass

### No Linter Currently

No ESLint or Prettier configuration exists. When adding:
- Prefer ESLint with TypeScript support
- Use Prettier for formatting

## Code Style Guidelines

### File Structure

- **Single-file architecture**: All plugin logic lives in `src/index.ts`
- **Output directory**: `dist/` (generated, gitignored)
- **Entry point**: `dist/index.js` with TypeScript declarations

### Imports

```typescript
// Use type-only imports for types
import type { Plugin } from "@opencode-ai/plugin"

// ESM module syntax only (no CommonJS)
export const AutoTitle: Plugin = async ({ client }) => { ... }
export default AutoTitle
```

### TypeScript Configuration

- **Target**: ES2022
- **Module**: ESNext with bundler resolution
- **Strict mode**: Enabled
- Generates declarations, declaration maps, and source maps

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Interfaces | PascalCase | `PluginConfig`, `State` |
| Functions | camelCase | `loadConfig`, `createLogger` |
| Constants | camelCase | `stopWords` |
| Environment vars | SCREAMING_SNAKE_CASE | `OPENCODE_AUTOTITLE_DEBUG` |
| Plugin export | PascalCase | `AutoTitle` |

### Function Patterns

```typescript
// Pure utility functions at module level
function sanitizeTitle(title: string, maxLength: number): string {
  return title
    .replace(/[^\w\s-]/g, "")
    .replace(/\s+/g, " ")
    .trim()
    .slice(0, maxLength)
}

// Async functions for SDK operations
async function generateAITitle(
  client: any,
  sessionId: string,
  userMessage: string,
  assistantMessage: string | null,
  config: PluginConfig,
  log: ReturnType<typeof createLogger>
): Promise<string | null> {
  // ...
}
```

### Error Handling

```typescript
// Use try-catch with graceful fallbacks
try {
  const result = await client.session.get({ path: { id: sessionId } })
  // ...
} catch (err) {
  log.error(`Failed: ${err instanceof Error ? err.message : "unknown"}`)
}

// Fire-and-forget cleanup with .catch(() => {})
await client.session.delete({ path: { id: tempSessionId } }).catch(() => {})
```

### Type Assertions

```typescript
// Use `as any` for dynamic SDK responses (SDK types are incomplete)
const sessionResponse = await client.session.get({
  path: { id: sessionId },
}) as any

const sessionData = sessionResponse?.data || sessionResponse
```

### Environment Variable Handling

```typescript
// All config via environment variables with OPENCODE_AUTOTITLE_ prefix
function loadConfig(): PluginConfig {
  const env = process.env
  return {
    model: env.OPENCODE_AUTOTITLE_MODEL || null,
    maxLength: Number(env.OPENCODE_AUTOTITLE_MAX_LENGTH) || 60,
    disabled: env.OPENCODE_AUTOTITLE_DISABLED === "1" || env.OPENCODE_AUTOTITLE_DISABLED === "true",
    debug: env.OPENCODE_AUTOTITLE_DEBUG || false,  // File path for debug logs
  }
}
```

### Logging Pattern

```typescript
// Create logger with debug flag and optional client
const log = createLogger(config.debug, client)

// Usage
log.debug("Detailed info")   // Only shows when OPENCODE_AUTOTITLE_DEBUG is set
log.info("Important info")   // Only shows when OPENCODE_AUTOTITLE_DEBUG is set
log.error("Always shows")    // Always outputs to stderr
```

### Plugin Export Pattern

```typescript
// Named export for explicit imports
export const AutoTitle: Plugin = async ({ client }) => {
  // Initialize config and state
  const config = loadConfig()
  const state: State = { titledSessions: new Set(), pendingSessions: new Set() }

  // Return event handler object
  return {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pawelma/opencode-autotitle](https://github.com/pawelma/opencode-autotitle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
