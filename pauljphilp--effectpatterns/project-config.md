---
trigger: always_on
description: Effect-Patterns is a community-driven knowledge base of practical patterns for building robust applications with Effect-TS. The project includes:
---

# Effect-Patterns AI Agent Instructions

## Project Overview

Effect-Patterns is a community-driven knowledge base of practical patterns for building robust applications with Effect-TS. The project includes:

- A pattern server for serving API endpoints
- Documentation and examples of Effect-TS patterns
- Rules and guidelines for AI coding agents
- MCP server integration for context-aware coding assistance

## Key Architectural Patterns

### Service Pattern

All services must use the modern `Effect.Service` pattern:

```typescript
export class UserService extends Effect.Service<UserService>()("UserService", {
  // Enable static accessor methods
  accessors: true,

  // Define implementation with dependencies
  effect: Effect.gen(function* () {
    // Get dependencies
    const logger = yield* LoggerService;
    const db = yield* DatabaseService;

    return {
      getUser: (id: string) =>
        Effect.gen(function* () {
          yield* logger.log(`Fetching user ${id}`);
          return yield* db.query(`SELECT * FROM users WHERE id = ${id}`);
        }),
    };
  }),

  // Declare dependencies
  dependencies: [LoggerService.Default, DatabaseService.Default],
}) {}
```

### Dependency Injection

- Use Layer-based DI with `Layer.merge`
- Compose layers logically:

```typescript
const mainLayer = Layer.merge(
  DatabaseService.Default,
  LoggerService.Default,
  NodeContext.layer
);

const program = Effect.gen(function* () {
  // Program logic
}).pipe(Effect.provide(mainLayer));
```

### Error Handling

Define tagged errors for type-safety:

```typescript
export class ServiceError extends Data.TaggedError("ServiceError")<{
  message: string;
  cause?: unknown;
}> {}

// Usage
Effect.gen(function* () {
  try {
    // Operation
  } catch (cause) {
    yield* Effect.fail(
      new ServiceError({
        message: "Operation failed",
        cause,
      })
    );
  }
});
```

### HTTP Server

Build HTTP servers with `@effect/platform`:

```typescript
const app = HttpRouter.empty.pipe(
  HttpRouter.get("/health", () =>
    Effect.succeed({ status: "ok" }).pipe(
      Effect.flatMap(HttpServerResponse.json)
    )
  )
);

const server = NodeHttpServer.layer(() => require("node:http").createServer(), {
  port: 3001,
});

const serverLayer = HttpServer.serve(app);
```

## Project Structure

```
/
├── api/           # API endpoint implementations
├── app/           # Next.js web application
├── content/       # Pattern documentation content
├── docs/         # Project documentation
├── packages/     # Shared packages
├── rules/        # AI coding rules
├── scripts/      # Build/deployment scripts
├── server/       # Pattern server implementation
└── services/     # Shared services
```

## Development Workflow

1. Start MCP server for AI assistance:
   ```bash
   npx @effect/mcp-server --layer src/layers.ts:AppLayer
   ```
2. Run server in dev mode:
   ```bash
   bun run mcp:dev
   ```
3. Run tests:
   ```bash
   bun test
   ```

## Testing Guidelines

### Test Structure

Place tests adjacent to implementation:

```
services/
  my-service/
    service.ts
    types.ts
    errors.ts
    __tests__/
      service.test.ts
```

### Test Pattern

```typescript
describe("MyService", () => {
  const testLayer = Layer.provide(MyService.Default, NodeContext.layer);

  it("should perform operation", () =>
    Effect.gen(function* () {
      const service = yield* MyService;
      const result = yield* service.myMethod("test");
      expect(result).toBe("expected");
    }).pipe(Effect.provide(testLayer)));

  it("should handle errors", () =>
    Effect.gen(function* () {
      const service = yield* MyService;
      const error = yield* service.riskyMethod().pipe(Effect.flip);
      expect(error).toBeInstanceOf(ServiceError);
    }).pipe(Effect.provide(testLayer)));
});
```

## Common Patterns

- Use `Effect.gen` for sequential operations
- Handle data validation with `Schema.struct()`
- Follow TypeScript strict mode conventions
- Use direct imports:

  ```typescript
  // ✅ Preferred
  import { Effect, Layer } from "effect";
  import { FileSystem } from "@effect/platform";

  // ❌ Avoid
  import * as Effect from "effect";
  ```

## Configuration & Deployment

### Configuration Pattern

Use type-safe configuration with Effect's Config service:

```typescript
// Define config schema
const ServerConfig = Config.nested("SERVER")(
  Config.all({
    host: Config.string("HOST"),
    port: Config.number("PORT"),
  })
);

// Create config service
class AppConfig extends Effect.Service<AppConfig>()("AppConfig", {
  effect: Effect.gen(function* () {
    const config = yield* ServerConfig;
    return {
      getConfig: () => Effect.succeed(config),
    };
  }),
}) {}

// Use in application
const program = Effect.gen(function* () {
  const config = yield* AppConfig;
  const { host, port } = yield* config.getConfig();
});
```

### Environment Setup

Required environment variables:

```env
# API Security
PATTERN_API_KEY=your-secret-api-key-here

# OpenTelemetry Configuration
OTLP_ENDPOINT=http://localhost:4318/v1/traces
OTLP_HEADERS=
SERVICE_NAME=effect-patterns-mcp-server

# Server Configuration
NODE_ENV=development
PORT=3000
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PaulJPhilp/EffectPatterns](https://github.com/PaulJPhilp/EffectPatterns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
