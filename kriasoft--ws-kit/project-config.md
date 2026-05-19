---
trigger: always_on
description: WS-Kit — Type-Safe WebSocket router for Bun and Cloudflare.
---

# WS-Kit

WS-Kit — Type-Safe WebSocket router for Bun and Cloudflare.

## Documentation

**ADRs** (`docs/adr/NNN-slug.md`): Architectural decisions (reference as ADR-NNN)
**SPECs** (`docs/specs/slug.md`): Component specifications (reference as docs/specs/slug.md)

### Component Specifications

- `docs/specs/README.md` — Navigation hub for all specifications
- `docs/specs/schema.md` — Message structure, type definitions, canonical imports
- `docs/specs/router.md` — Server router API, handlers, lifecycle hooks
- `docs/specs/validation.md` — Validation flow, normalization, error handling
- `docs/specs/context-methods.md` — Handler context methods: send, reply, progress, publish
- `docs/specs/plugins.md` — Plugin system: core plugins, adapters, custom plugins
- `docs/specs/canonical-imports.md` — Quick reference for canonical import sources
- `docs/specs/pubsub.md` — Pub/Sub API, topic subscriptions, publishing, patterns
- `docs/specs/client.md` — Client SDK API, connection states, queueing
- `docs/specs/adapters.md` — Platform adapter responsibilities, limits, pub/sub guarantees
- `docs/specs/patterns.md` — Architectural patterns for production applications
- `docs/specs/rules.md` — Development rules (MUST/NEVER) with links to details
- `docs/specs/error-handling.md` — Error codes and patterns
- `docs/specs/test-requirements.md` — Type-level and runtime test requirements
- `docs/adr/README.md` — Architectural decision records index

## Architecture

- **Modular Packages**: `@ws-kit/core` router with pluggable validator and platform adapters
- **Capability Plugins**: `.plugin()` gates features (validation, pub/sub) for both runtime and types
- **Composition Over Inheritance**: Single `WebSocketRouter<V>` class, any validator + platform combo works
- **Message-Based Routing**: Routes by message `type` field to registered handlers
- **Type Safety**: Full TypeScript inference from schema to handler via generics and overloads
- **Platform Adapters**: `@ws-kit/bun`, `@ws-kit/cloudflare`, etc. each with both high-level and low-level APIs
- **Validator Adapters**: `@ws-kit/zod`, `@ws-kit/valibot`, custom validators welcome via `ValidatorAdapter` interface

## API Design Principles

- **Plain functions**: `message()` and `createRouter()` are plain functions, not factories
- **Full type inference**: TypeScript generics preserve types from schema through handlers without assertions
- **Runtime identity**: Functions preserve `instanceof` checks and runtime behavior

## Quick Start

```typescript
import { z, message, createRouter, withZod } from "@ws-kit/zod";
import { withPubSub } from "@ws-kit/pubsub";
import { redisPubSub } from "@ws-kit/redis";
import { serve } from "@ws-kit/bun";
import { createClient } from "redis";

declare module "@ws-kit/core" {
  interface ConnectionData {
    userId?: string;
    roles?: string[];
  }
}

const redis = createClient({ url: process.env.REDIS_URL! });
await redis.connect();

const PingMessage = message("PING", { text: z.string() });
const PongMessage = message("PONG", { reply: z.string() });

const router = createRouter()
  .plugin(withZod())
  .plugin(withPubSub({ adapter: redisPubSub(redis) }));

router.on(PingMessage, (ctx) => {
  ctx.send(PongMessage, { reply: `Got: ${ctx.payload.text}` });
});

serve(router, {
  port: 3000,
  authenticate(req) {
    const token = req.headers.get("authorization");
    return token ? { userId: "user_123", roles: ["admin"] } : undefined;
  },
});
```

**Key concepts**:

- **Canonical imports** (see ADR-032): Always import plugins from their official sources
  - Validators + helpers: `@ws-kit/zod` or `@ws-kit/valibot` (choose one)
  - Core plugins: `@ws-kit/plugins` (`withMessaging`, `withRpc`)
  - Feature plugins: Feature-specific packages (`@ws-kit/pubsub`, `@ws-kit/rate-limit`, etc.)
  - Adapters: Adapter packages (`@ws-kit/memory`, `@ws-kit/redis`, `@ws-kit/cloudflare`)
- Plugins add capabilities: `.plugin(withZod())` (validation), `.plugin(withPubSub())` (broadcasting)
- Module augmentation for `ConnectionData` (define once, shared across routers)
- Adapters provide backends: memory (dev), Redis/Cloudflare (production)

## API Surface

All available methods at a glance:

```typescript
// Fire-and-forget messaging
router.on(Message, async (ctx) => {
  ctx.send(schema, data); // Send to current connection (1-to-1)
  ctx.publish(topic, schema, data); // Broadcast to topic subscribers (1-to-many)
  await ctx.topics.subscribe(topic); // Subscribe to topic (async)
  await ctx.topics.unsubscribe(topic); // Unsubscribe from topic (async)
  ctx.data; // Access typed connection data
  ctx.assignData(partial); // Update connection data
});

// Request-response pattern (RPC)
router.rpc(Request, (ctx) => {
  ctx.reply(payload, opts?); // Terminal response (one-shot)
  ctx.progress(update, opts?); // Non-terminal progress updates
});

// Client-side
client.send(schema, data); // Fire-and-forget to server
client.request(schema, data); // RPC call (returns Promise, auto-correlation)
```

**Naming rationale** (see ADRs):

- `send()` vs `publish()` — one connection vs many (ADR-020)
- `reply()` vs `send()` — RPC terminal response vs fire-and-forget (ADR-015)
- `progress()` — non-terminal RPC updates for streaming (ADR-015)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kriasoft/ws-kit](https://github.com/kriasoft/ws-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
