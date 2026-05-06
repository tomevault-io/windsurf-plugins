---
trigger: always_on
description: | Command | Description |
---

# AGENTS.md

## Available Scripts

| Command | Description |
|---------|-------------|
| `bun run typecheck` | Run TypeScript type checking without emitting files |
| `bun run lint` | Run ESLint on the project |
| `bun run lint:fix` | Run ESLint and automatically fix issues |
| `bun run format` | Format code using ESLint |
| `bun run test` | Run tests once with Vitest |
| `bun run test:watch` | Run tests in watch mode |

---

## Effect Repository Reference (.context/effect-repo)

The Effect repository is available as a subtree for reference patterns and best practices.

### Repository Structure

```
.context/effect-repo/packages/
├── effect/                    # Core Effect library
├── platform/                  # Platform abstractions (HTTP, FileSystem, etc.)
├── platform-node/             # Node.js implementation
├── platform-bun/              # Bun implementation
├── platform-browser/          # Browser implementation
├── rpc/                       # RPC (Remote Procedure Call) framework
├── sql/                       # SQL client abstractions
├── sql-pg/, sql-sqlite-*/     # Database-specific SQL implementations
├── cli/                       # CLI framework
├── vitest/                    # Vitest testing utilities
├── ai/                        # AI integrations
├── cluster/                   # Clustering support
├── workflow/                  # Workflow/orchestration
├── experimental/              # Experimental features
└── opentelemetry/             # OpenTelemetry integration
```

### Key Packages

| Package | Description |
|---------|-------------|
| `effect` | Core library: Effect, Schema, Stream, Layer, Context |
| `@effect/platform` | Platform-agnostic: HttpClient, FileSystem, etc. |
| `@effect/rpc` | Type-safe RPC framework |
| `@effect/sql` | SQL client abstractions |
| `@effect/vitest` | Testing utilities for Vitest |

### Module Structure Pattern

```
packages/<pkg>/src/
├── ModuleName.ts        # Public API (re-exports + docs)
├── internal/
│   └── moduleName.ts    # Internal implementation
└── index.ts             # Package entry point
```

Internal modules (`internal/`) contain implementations; public modules re-export with documentation.

---

## Key File Locations

### Core Effect Types
| Path | Description |
|------|-------------|
| `.context/effect-repo/packages/effect/src/Effect.ts` | Effect type and functions |
| `.context/effect-repo/packages/effect/src/internal/core.ts` | Core primitives |
| `.context/effect-repo/packages/effect/src/Layer.ts` | Dependency injection |
| `.context/effect-repo/packages/effect/src/Context.ts` | Context and Tags |
| `.context/effect-repo/packages/effect/src/Cause.ts` | Error cause model |
| `.context/effect-repo/packages/effect/src/Exit.ts` | Exit type |

### Schema
| Path | Description |
|------|-------------|
| `.context/effect-repo/packages/effect/src/Schema.ts` | Schema DSL |
| `.context/effect-repo/packages/effect/src/SchemaAST.ts` | Schema AST |
| `.context/effect-repo/packages/effect/src/ParseResult.ts` | Parse results |

### Platform/HTTP
| Path | Description |
|------|-------------|
| `.context/effect-repo/packages/platform/src/HttpClient.ts` | HTTP client |
| `.context/effect-repo/packages/platform/src/HttpServer.ts` | HTTP server |
| `.context/effect-repo/packages/platform/src/HttpRouter.ts` | HTTP router |
| `.context/effect-repo/packages/platform/src/HttpApi.ts` | HTTP API definition |
| `.context/effect-repo/packages/platform/src/HttpApiBuilder.ts` | API builder |

### RPC
| Path | Description |
|------|-------------|
| `.context/effect-repo/packages/rpc/src/Rpc.ts` | RPC procedure definition |
| `.context/effect-repo/packages/rpc/src/RpcGroup.ts` | Grouping RPCs |
| `.context/effect-repo/packages/rpc/src/RpcClient.ts` | RPC client |
| `.context/effect-repo/packages/rpc/src/RpcServer.ts` | RPC server |

---

## Code Patterns

### Service/Tag Definition (Preferred Pattern)

```typescript
import * as Context from "effect/Context"
import * as Layer from "effect/Layer"

export class MyService extends Context.Tag("@myorg/pkg/MyService")<
  MyService,
  MyService.Service
>() {
  static readonly Live = Layer.effect(MyService, Effect.gen(function*() {
    // implementation
  }))
}

export namespace MyService {
  export interface Service {
    readonly doSomething: (input: string) => Effect.Effect<Result, MyError>
  }
}
```

### Error Definition Patterns

**Data.TaggedError (most common):**
```typescript
import * as Data from "effect/Data"

export class MyError extends Data.TaggedError("MyError")<{
  readonly message: string
  readonly cause?: unknown
}> {}
```

**Schema.TaggedError (serializable):**
```typescript
import * as Schema from "effect/Schema"

export class ApiError extends Schema.TaggedError<ApiError>()(
  "ApiError",
  { status: Schema.Number, message: Schema.String }
) {}
```

### Type ID Pattern

```typescript
export const TypeId: unique symbol = Symbol.for("@myorg/pkg/MyType")
export type TypeId = typeof TypeId

export interface MyType {
  readonly [TypeId]: TypeId
  // ...
}
```

### Layer Definition

```typescript
// Simple layer
const Live = Layer.succeed(MyService, implementation)

// Effect-based layer
const Live = Layer.effect(MyService, Effect.gen(function*() {
  const dep = yield* SomeDependency
  return { /* implementation */ }
}))


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikearnaldi/effect-trpc](https://github.com/mikearnaldi/effect-trpc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
