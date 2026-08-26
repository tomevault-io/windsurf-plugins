---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ClaudeBench is a Redis-first event-driven system where every handler auto-generates HTTP, MCP, and event interfaces through a single decorator. The architecture enforces localhost-reality: no distributed complexity for single-user tools.

## Critical Commands

```bash
bun relay               # MUST run in background - monitors all system events
bun dev                 # Start server (:3000) and web (:3001)
bun test:contract       # Test external API contracts (specs/001-claudebench/contracts/)
bun test:integration    # Test internal Redis/Prisma side effects
```

## Codebase Architecture

### Handler-Centric Design

Every feature is a handler in `apps/server/src/handlers/{domain}/`. The handler IS the feature:

```typescript
// apps/server/src/handlers/task/task.create.handler.ts
@EventHandler({
  event: 'task.create',              // Becomes HTTP POST /task/create
  inputSchema: taskCreateInput,      // Shared schema from schemas/
  outputSchema: taskCreateOutput,    // Type-safe everywhere
  persist: true,                     // Handler decides PostgreSQL persistence
  rateLimit: 10
})
export class TaskCreateHandler {
  @Instrumented(0)                   // Caching TTL (0 = no cache)
  @Resilient({                       // Per-handler resilience config
    rateLimit: { limit: 100, windowMs: 60000 },
    timeout: 5000,
    circuitBreaker: { threshold: 5, timeout: 30000 }
  })
  async handle(input: TaskCreateInput, ctx: EventContext) {
    // 1. Atomic Redis operations via Lua scripts
    const result = await redisScripts.createTask(...);
    
    // 2. Conditional PostgreSQL persistence
    if (ctx.persist) {
      await ctx.prisma.task.create({ data });
    }
    
    // 3. Publish events for observers
    await ctx.publish({ type: 'task.created', payload });
    
    return output; // Validated by outputSchema
  }
}
```

### Directory Structure Patterns

```
apps/server/src/
├── core/
│   ├── decorator.ts       # @EventHandler, @Instrumented, @Resilient
│   ├── context.ts         # EventContext with Redis, Prisma, publish
│   ├── redis-scripts.ts   # Lua scripts for atomic operations
│   └── bus.ts             # Event bus initialization
├── handlers/
│   ├── task/              # Task domain
│   │   ├── task.create.handler.ts
│   │   ├── task.complete.handler.ts
│   │   └── index.ts       # Exports all handlers
│   ├── swarm/             # Swarm intelligence
│   └── system/            # System operations
├── schemas/               # Shared Zod schemas
│   ├── task.schema.ts     # Input/Output types for task domain
│   └── common.schema.ts   # Shared types
└── transports/
    ├── http.ts            # Auto-generated from decorators
    └── mcp.ts             # Auto-generated MCP tools
```

## Key Architectural Patterns

### 1. Redis Lua Scripts (Atomic Operations)

All Redis operations use Lua scripts for atomicity (`core/redis-scripts.ts`):

```typescript
// Instead of multiple Redis calls:
// ❌ await redis.hset(); await redis.zadd(); await redis.incr();

// Use atomic Lua script:
// ✅ await redisScripts.createTask(taskId, text, priority, status, now, metadata);
```

Lua scripts handle:
- Task creation with queue addition
- Atomic metrics updates
- Conflict detection
- State transitions

### 2. EventContext Pattern

Every handler receives `EventContext` with unified access:

```typescript
interface EventContext {
  instanceId: string;        // Worker identity
  requestId: string;         // Trace requests
  redis: RedisConnection;    // Direct Redis access
  prisma: PrismaClient;      // Direct DB access
  persist: boolean;          // From decorator config
  publish: (event) => void;  // Emit events
  metrics: MetricsClient;    // Prometheus metrics
}
```

### 3. Schema-First Development

Schemas define the contract (`schemas/*.schema.ts`):

```typescript
// schemas/task.schema.ts
export const taskCreateInput = z.object({
  text: z.string().min(1).max(500),
  priority: z.number().min(0).max(100).optional(),
  metadata: z.record(z.unknown()).optional()
});

export type TaskCreateInput = z.infer<typeof taskCreateInput>;
```

Schemas are:
- Shared between handlers and tests
- Used for validation at all boundaries
- The source of TypeScript types
- Never duplicated between transports

## Testing Philosophy

### Contract Tests (External Behavior)

Location: `apps/server/tests/contract/`
Purpose: Verify API contracts match specifications

```typescript
// Tests against specs/001-claudebench/contracts/jsonrpc-contract.json
it('should create task with correct shape', async () => {
  const response = await callRPC('task.create', { text: 'Test' });
  expect(response).toMatchContract('task.create.output');
});
```

### Integration Tests (Internal Behavior)

Location: `apps/server/tests/integration/`
Purpose: Verify Redis keys, queues, and side effects

```typescript
it('should add task to Redis queue', async () => {
  await handler.handle(input, ctx);
  
  // Verify internal state changes
  const queueLength = await redis.zcard('cb:queue:tasks');
  expect(queueLength).toBe(1);
  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fblgit/claudebench](https://github.com/fblgit/claudebench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
