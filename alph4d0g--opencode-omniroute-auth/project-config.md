---
trigger: always_on
description: This file provides guidelines to AI agents when working with code in this repository.
---

# Agent Guidelines for opencode-omniroute-auth

This file provides guidelines to AI agents when working with code in this repository.

## Overview

`opencode-omniroute-auth` is an OpenCode authentication plugin for the OmniRoute API. It provides a `/connect omniroute` command, API-key auth, dynamic model fetching from `/v1/models`, and combo model capability enrichment.

## Common Commands

```bash
# Build (required before running tests)
npm run build

# Watch mode during development
npm run dev

# Run tests (builds first, then runs Node built-in test runner)
npm test

# Run a single test file
npm run build && node --test test/plugin.test.mjs

# Type-check a single file without emitting
npx tsc --noEmit src/plugin.ts

# Clean build output
npm run clean

# Validate dist exports satisfy plugin loader constraints
npm run check:exports

# Full publish prep
npm run prepublishOnly
```

## Architecture

### Dual Entry Points

- **`index.ts`** — Main plugin export (`OmniRouteAuthPlugin`). Required by OpenCode's plugin loader. All root exports must be functions.
- **`runtime.ts`** — Runtime utilities (`fetchModels`, `clearModelCache`, combo helpers, etc.) exported for programmatic use.

### Core Modules

| File | Responsibility |
|------|----------------|
| `src/plugin.ts` | Plugin implementation: `config` hook (registers `omniroute` provider), `auth` hook (`/connect` command), `loadProviderOptions` (fetches models and returns a `fetch` interceptor). |
| `src/models.ts` | `fetchModels()` fetches `/v1/models`, manages an in-memory cache keyed by `baseUrl:apiKey`, falls back to defaults on failure. Orchestrates metadata enrichment via `models-dev.ts` and combo enrichment via `omniroute-combos.ts`. |
| `src/models-dev.ts` | Fetches `https://models.dev/api.json`, builds indexed lookup maps (exact/normalized, provider-specific and global), and maps OmniRoute provider keys to models.dev providers via aliases. |
| `src/omniroute-combos.ts` | Fetches combo definitions from `/api/combos`. Resolves underlying models and calculates lowest-common-denominator capabilities (min context/maxTokens, vision/tools only if ALL underlying models support them). |
| `src/constants.ts` | Endpoints, default models, TTLs, timeouts. |
| `src/types.ts` | Shared TypeScript interfaces. |

### Fetch Interceptor (`createFetchInterceptor` in `src/plugin.ts`)

The loader returns a `fetch` function that:
1. Adds `Authorization: Bearer <apiKey>` and `Content-Type: application/json` headers.
2. Only intercepts requests to the configured OmniRoute base URL (with safe prefix matching).
3. Sanitizes Gemini tool schemas by stripping `$schema`, `$ref`, `ref`, and `additionalProperties` keywords when the model name includes "gemini".

### Caching Strategy

Three independent in-memory caches:
- **Model cache** (`src/models.ts`) — keyed by `baseUrl:apiKey`, TTL defaults to 5 minutes.
- **models.dev cache** (`src/models-dev.ts`) — global singleton, TTL defaults to 24 hours.
- **Combo cache** (`src/omniroute-combos.ts`) — global singleton, TTL defaults to 5 minutes.

`clearModelCache()` also clears the combo cache.

## Code Style Guidelines

### TypeScript & Formatting
- **Target**: ES2022, **Module**: NodeNext (ESM).
- **Strict Mode**: Enabled. Never disable strict checks.
- **Formatting**: 2 spaces, max 100 chars/line, semicolons required, **single quotes** for strings, trailing commas in multi-line objects/arrays.

### Naming Conventions
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `OMNIROUTE_PROVIDER_ID`)
- **Variables/Functions**: `camelCase` (e.g., `modelCache`, `fetchModels()`)
- **Classes/Interfaces/Types**: `PascalCase` (e.g., `OmniRouteConfig`)
- **Files**: `kebab-case` (e.g., `opencode-plugin.d.ts`)

### Imports
- **CRITICAL**: Always use explicit `.js` extensions for relative imports (e.g., `import { x } from './file.js'`).
- Group imports: external → internal → types.
- Use named exports only (no default exports).

### Type Safety
- **Never use `any`**. Use `unknown` if uncertain, then narrow.
- Always type function parameters and return types.
- **Prefer runtime validation** over unsafe type assertions.
```typescript
// ✅ Correct
const rawData = await response.json();
if (!rawData || typeof rawData !== 'object' || !Array.isArray(rawData.data)) {
  throw new Error('Invalid response structure');
}
const data = rawData as OmniRouteModelsResponse;
```

### Error Handling & Logging
- Always use `try/catch/finally` for resource cleanup (e.g., `clearTimeout`).
- Provide meaningful error messages.
- **Security**: Sanitize error logs. Never log full API responses or sensitive keys (e.g., log "Cache cleared for provided config" instead of logging the API key).
```typescript
// ✅ Correct
const controller = new AbortController();
const timeoutId = setTimeout(() => controller.abort(), REQUEST_TIMEOUT);
try {
  const response = await fetch(url, { signal: controller.signal });
  if (!response.ok) throw new Error(`Request failed: ${response.status}`);
  return await response.json();
} finally {
  clearTimeout(timeoutId);
}
```

### Headers & URL Handling
- **Headers**: Use the `Headers` constructor for proper normalization.
  ```typescript
  const headers = new Headers(init?.headers);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Alph4d0g/opencode-omniroute-auth](https://github.com/Alph4d0g/opencode-omniroute-auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
