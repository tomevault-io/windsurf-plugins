---
trigger: always_on
description: Guidelines for AI agents and human contributors working on this codebase.
---

# AGENTS.md

Guidelines for AI agents and human contributors working on this codebase.

## Project Overview

This is an MCP server that delegates coding tasks to OpenCode running in Cloudflare Sandboxes. It's built on Cloudflare Workers with Durable Objects, Workflows, Containers, and R2.

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     MCP Client (Claude, etc.)                    │
└────────────────────────────┬────────────────────────────────────┘
                             │ MCP Protocol
┌────────────────────────────▼────────────────────────────────────┐
│                   Cloudflare Worker                              │
│  Routes: /mcp, /proxy/*, /session/{id}/                          │
└────────────────────────────┬────────────────────────────────────┘
         │                   │                    │
    ┌────▼────┐       ┌──────▼──────┐      ┌─────▼─────┐
    │ Proxy   │       │ MCP Agent   │      │ Web UI    │
    │ Handler │       │    (DO)     │      │ Proxy     │
    └────┬────┘       └──────┬──────┘      └─────┬─────┘
         │                   │                    │
         │           ┌───────▼───────┐           │
         │           │   Workflow    │           │
         │           └───────┬───────┘           │
         │                   │                   │
    ┌────▼───────────────────▼───────────────────▼────┐
    │                 Cloudflare Sandbox               │
    │   ┌─────────────────────────────────────────┐   │
    │   │           OpenCode Agent                 │   │
    │   │  - Autonomous coding                     │   │
    │   │  - Git operations                        │   │
    │   │  - Web UI on port 4096                   │   │
    │   └─────────────────────────────────────────┘   │
    └─────────────────────────────────────────────────┘
```

### Key Components

| Component | Purpose |
|-----------|---------|
| **Worker** | HTTP routing, MCP endpoint, proxy |
| **MCP Agent (DO)** | Protocol handling, tool implementation |
| **Workflow** | Long-running task orchestration (up to 50min) |
| **Sandbox** | Isolated container running OpenCode |
| **R2** | Session metadata, workspace persistence |

## Tech Stack

- **Runtime**: Cloudflare Workers
- **Language**: TypeScript (strict mode)
- **Framework**: Effect for services, errors, and dependency injection
- **Testing**: Vitest
- **Linting**: Oxlint
- **Formatting**: Oxfmt

## Code Style

### Effect Patterns

Services use Effect's Context/Layer pattern:

```typescript
// Define service interface
interface MyServiceInterface {
  readonly doThing: (id: string) => Effect.Effect<Result, MyError>;
}

// Create Context tag
export class MyService extends Context.Tag("MyService")<
  MyService,
  MyServiceInterface
>() {}

// Create Layer factory
export function makeMyServiceLayer(deps: Deps): Layer.Layer<MyService> {
  return Layer.succeed(MyService, makeMyServiceImpl(deps));
}

// Use in Effect.gen
const program = Effect.gen(function* () {
  const service = yield* MyService;
  return yield* service.doThing("123");
});

// Provide layer and run
Effect.runPromise(Effect.provide(program, layer));
```

### Error Types

Use `Schema.TaggedError` for typed errors:

```typescript
export class MyError extends Schema.TaggedError<MyError>()("MyError", {
  id: Schema.String,
  cause: Schema.String,
}) {
  override get message(): string {
    return `Failed for ${this.id}: ${this.cause}`;
  }
}
```

### Schemas

Use Effect Schema for data validation:

```typescript
const MyModel = Schema.Struct({
  id: Schema.String,
  status: Schema.Literal("active", "stopped"),
  createdAt: Schema.Number,
});
type MyModel = typeof MyModel.Type;
```

For MCP tools, use Zod (required by MCP SDK):

```typescript
export const myToolInputSchema = {
  type: "object",
  properties: {
    id: { type: "string", description: "The ID" },
  },
  required: ["id"],
} as const;
```

## Architecture Decisions

### Cloudflare Bindings

Configured in `wrangler.jsonc`:

| Binding | Type | Purpose |
|---------|------|---------|
| `MCP_AGENT` | Durable Object | MCP protocol handler |
| `Sandbox` | Durable Object | Container instances |
| `SESSIONS_BUCKET` | R2 Bucket | Session/workspace storage |
| `EXECUTE_TASK_WORKFLOW` | Workflow | Task execution |

### Storage Layout

All data is stored in R2 for cross-DO access. Key patterns are defined in `src/storage/keys.ts`:

```
sessions/_index.json           # Global session index
sessions/{sessionId}.json      # Session metadata (flat)
runs/_index.json               # Global runs index
runs/{runId}.json              # Run data (flat)
```

- **Sessions in R2**: Cross-DO access (MCP lib creates separate DO per connection)
- **Runs in R2**: Global index enables cross-session queries without knowing sessionIds

### Zero-Trust Proxy

Real credentials never enter sandbox. Proxy validates JWT, injects real creds:

```
Sandbox → JWT as API key → Proxy → Real credentials → External API
```

**Provider support:** Currently hardcoded to Anthropic. To add other providers, see the [Customizing the Provider](./README.md#customizing-the-provider) section in README. The key files are:
- `src/proxy/services/` - Service configs (target URL, auth injection)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghostwriternr/sandbox-mcp](https://github.com/ghostwriternr/sandbox-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
