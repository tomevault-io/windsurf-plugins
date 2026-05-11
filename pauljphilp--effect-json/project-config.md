---
trigger: always_on
description: Development guidelines and context for effect-json library
---


# effect-json Development Guide

**effect-json** is a published npm library providing type-safe, schema-driven JSON serialization for TypeScript and Effect.

📦 Published: https://www.npmjs.com/package/effect-json
🐙 Repository: https://github.com/PaulJPhilp/effect-json
📊 Version: 0.1.0 (production)

## Project Overview

effect-json provides three JSON serialization backends unified under a single Effect-native API:
- **JSON**: Standard JSON.parse/stringify with precise error reporting
- **JSONC**: JSON with Comments support (strips single-line and multi-line comments)
- **SuperJSON**: Type-preserving serialization (Date, Set, Map, BigInt, etc.)

All operations return Effect types for composability with the Effect ecosystem.

## Architecture

### Core Principles

1. **Effect-First Design**: All APIs return `Effect<Success, Error>` types
2. **Tagged Errors**: Use `ParseError`, `ValidationError`, `StringifyError` for precise error handling
3. **Schema Validation**: Effect.Schema for all parsing/stringification
4. **Pluggable Backends**: Backend interface allows extensibility
5. **Type Safety**: TypeScript strict mode, full type inference

### Project Structure

```
effect-json/
├── packages/effect-json/          # Main package
│   ├── src/
│   │   ├── api.ts                 # Public API (parse, stringify, etc.)
│   │   ├── backends/              # JSON, JSONC, SuperJSON implementations
│   │   │   ├── types.ts           # Backend interface
│   │   │   ├── json.ts            # Standard JSON backend
│   │   │   ├── jsonc.ts           # JSON with Comments
│   │   │   └── superjson.ts       # Type-preserving backend
│   │   ├── errors.ts              # Tagged error types
│   │   ├── schema.ts              # Schema validation utilities
│   │   ├── config.ts              # Service layer (DI)
│   │   ├── utils/                 # String utilities
│   │   └── __tests__/             # Test suites
│   ├── dist/                      # Built output (gitignored)
│   └── package.json
├── docs/                          # Planning documentation
└── .github/workflows/             # CI/CD pipelines
```

## Development Standards

### Runtime & Tools

**Default to using Bun instead of Node.js:**

- Use `bun <file>` instead of `node <file>` or `ts-node <file>`
- Use `bun test` instead of `jest` or `vitest`
- Use `bun install` instead of `npm install`
- Use `bun run <script>` instead of `npm run <script>`
- Bun automatically loads .env, don't use dotenv

### Effect Best Practices

1. **Use Effect.gen for composition:**
   ```typescript
   export const parse = <A, I>(schema: Schema.Schema<A, I>, input: string) =>
     Effect.gen(function* () {
       const raw = yield* backend.parse(input);
       const validated = yield* validateAgainstSchema(schema, raw);
       return validated;
     });
   ```

2. **Always use tagged errors:**
   ```typescript
   export class ParseError extends Data.TaggedError("ParseError")<{
     readonly message: string;
     readonly line: number;
     readonly column: number;
     readonly snippet: string;
   }> {}
   ```

3. **Leverage Effect.try for external APIs:**
   ```typescript
   Effect.try({
     try: () => JSON.parse(input),
     catch: (error) => new ParseError({ ... }),
   });
   ```

4. **Use pipe for transformations:**
   ```typescript
   Schema.decode(schema)(data).pipe(
     Effect.mapError((parseError) => new ValidationError({ ... })),
   );
   ```

### Code Style

- **TypeScript strict mode**: Enabled in tsconfig.json
- **Biome formatting**: Run `bun run check` before committing
- **No unused variables**: Prefix with `_` if intentionally unused
- **JSDoc comments**: Required for all public APIs
- **Immutability**: Prefer `readonly` and avoid mutations
- **Small functions**: Keep functions focused and testable

### Testing Requirements

- **Test location**: `src/__tests__/` directory
- **Test runner**: Vitest (via `bun test`)
- **Coverage target**: Minimum 85% (currently at 85.62%)
- **Test structure**:
  - `unit/backends/` - Backend-specific tests
  - `integration/` - Cross-backend, roundtrip, error recovery
  - `golden.test.ts` - Fixture-based tests

**Example test pattern:**
```typescript
import { Effect, Schema } from "effect";
import { describe, expect, it } from "vitest";
import * as Json from "../index.js";

describe("Feature", () => {
  it("should handle success case", async () => {
    const schema = Schema.Struct({ id: Schema.Number });
    const result = await Effect.runPromise(
      Json.parse(schema, '{"id": 1}')
    );
    expect(result).toEqual({ id: 1 });
  });

  it("should handle error case", async () => {
    const result = await Effect.runPromise(
      Effect.either(Json.parse(schema, 'invalid'))
    );
    expect(result._tag).toBe("Left");
    expect(result.left._tag).toBe("ParseError");
  });
});
```

### Backend Implementation

When adding new backends, implement the `Backend` interface:

```typescript
export interface Backend {
  readonly parse: (input: string | Buffer) => Effect.Effect<unknown, ParseError>;
  readonly stringify: (value: unknown, options?: StringifyOptions) =>
    Effect.Effect<string, StringifyError>;
}
```

**Backend guidelines:**
- Delegate to existing backends when possible (see JSONC)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PaulJPhilp/effect-json](https://github.com/PaulJPhilp/effect-json) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
