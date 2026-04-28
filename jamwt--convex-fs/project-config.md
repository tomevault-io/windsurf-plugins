---
trigger: always_on
description: This document provides guidelines for AI agents working on the ConvexFS
---

# AGENTS.md - Guidelines for AI Coding Agents

This document provides guidelines for AI agents working on the ConvexFS
codebase.

## Project Overview

ConvexFS is a Convex component providing virtual filesystem semantics backed by
external blob storage (S3-compatible or Bunny.net). It's structured as:

- `src/component/` - Convex backend (queries, mutations, actions, schema)
- `src/client/` - Client SDK (`ConvexFS` class, `registerRoutes`)
- `src/react/` - React hooks
- `example/` - Demo app with Vite frontend + Convex backend

## Build/Lint/Test Commands

```bash
# Install dependencies
npm install

# Development (runs backend, frontend, and build watcher)
npm run dev

# Build TypeScript
npm run build
npm run build:clean    # Clean rebuild with codegen

# Type checking
npm run typecheck

# Linting
npm run lint

# Run all tests with type checking
npm test

# Run tests in watch mode
npm run test:watch

# Run a single test file
npx vitest run src/component/blobstore/s3.test.ts

# Run tests matching a pattern
npx vitest run -t "put"

# Run a single test by name
npx vitest run -t "sends PUT request with presigned URL"

# Debug tests
npm run test:debug

# Regenerate Convex types
npx convex dev --once
```

## Code Style Guidelines

### Formatting

- **Prettier** with trailing commas (`"trailingComma": "all"`)
- **2 spaces** for indentation (TypeScript default)
- Run `npx prettier -w <file>` to format

### Imports

1. External packages first (convex, vitest, etc.)
2. Internal absolute imports second
3. Relative imports last
4. Use `.js` extension for relative imports (ESM requirement)
5. Separate `import type` from value imports

```typescript
// External
import { v } from "convex/values";
import { describe, it, expect } from "vitest";

// Internal - types separate
import type { BlobMetadata } from "./blobstore/index.js";
import { createBlobStore } from "./blobstore/index.js";

// Relative
import { configValidator } from "./validators.js";
```

### TypeScript

- **Strict mode** enabled - no implicit any, strict null checks
- Use `type` keyword for type-only imports: `import type { Foo } from ...`
- Prefer interfaces for object shapes, types for unions/intersections
- Use `Infer<typeof validator>` to derive types from Convex validators
- Prefix unused parameters with underscore: `_ctx`, `_opts`

```typescript
// Deriving types from validators
export const configValidator = v.object({ ... });
export type Config = Infer<typeof configValidator>;

// Unused parameters
async generateUploadUrl(_key: string, _opts?: UploadUrlOptions): Promise<string>
```

### Naming Conventions

- **Files**: `camelCase.ts` for modules, `camelCase.test.ts` for tests
- **Classes**: `PascalCase` (e.g., `ConvexFS`, `BlobStore`)
- **Functions/methods**: `camelCase` (e.g., `createBlobStore`, `getDownloadUrl`)
- **Constants**: `UPPER_SNAKE_CASE` for true constants, `camelCase` otherwise
- **Types/Interfaces**: `PascalCase` (e.g., `BlobMetadata`, `StorageConfig`)
- **Validators**: `camelCaseValidator` suffix (e.g., `configValidator`)

### Convex Patterns

```typescript
// Queries/mutations/actions export pattern
export const myQuery = query({
  args: { path: v.string() },
  returns: v.union(v.null(), fileMetadataValidator),
  handler: async (ctx, args) => { ... },
});

// Internal functions use internalQuery/internalMutation/internalAction
export const myInternalQuery = internalQuery({ ... });

// Schema definition
export default defineSchema({
  tableName: defineTable({
    field: v.string(),
  }).index("indexName", ["field"]),
});
```

### Error Handling

- Throw `Error` with descriptive messages for expected failures
- Use try/catch in HTTP handlers, return appropriate status codes
- Log errors with `console.error()` before returning error responses

```typescript
// In actions/mutations
if (!config) {
  throw new Error("Storage not configured");
}

// In HTTP handlers
try {
  const result = await doSomething();
  return new Response(JSON.stringify(result), { status: 200 });
} catch (error) {
  console.error("Operation failed:", error);
  return new Response("Error message", { status: 500 });
}
```

### Testing Patterns

- Use Vitest with `describe`/`it`/`expect`
- Mock `globalThis.fetch` for HTTP tests
- Use `beforeEach`/`afterEach` for setup/teardown
- Test files adjacent to source: `foo.ts` -> `foo.test.ts`

```typescript
describe("createS3BlobStore", () => {
  let originalFetch: typeof globalThis.fetch;

  beforeEach(() => {
    originalFetch = globalThis.fetch;
  });

  afterEach(() => {
    globalThis.fetch = originalFetch;
    vi.restoreAllMocks();
  });

  it("does something", async () => {
    const mockFetch = vi.fn().mockResolvedValue(new Response(...));
    globalThis.fetch = mockFetch;
    // test logic
    expect(mockFetch).toHaveBeenCalledTimes(1);
  });
});
```

### Documentation

- Use JSDoc comments for public APIs with `@example` blocks
- Document complex parameters with `@param`
- Keep inline comments brief and focused on "why" not "what"

## Project-Specific Patterns

### BlobStore Interface

All storage backends implement the `BlobStore` interface:

```typescript
interface BlobStore {
  generateUploadUrl(key: string, opts?: UploadUrlOptions): Promise<string>;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jamwt/convex-fs](https://github.com/jamwt/convex-fs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
