---
trigger: always_on
description: When working with elysiaJS, here are reference guides:
---

When working with elysiaJS, here are reference guides:

## ⚠️ CRITICAL: OpenAPI Response Schemas

**ALWAYS use status-code keyed objects for response schemas, NOT `t.Union()`.**

This ensures OpenAPI documentation properly displays all response parameters.

### ❌ WRONG - Response won't show in OpenAPI docs:
```typescript
{
  response: t.Union([SuccessResponse, ErrorResponse]),
}
```

### ✅ CORRECT - All responses properly documented:
```typescript
{
  response: {
    200: SuccessResponse,        // Success
    201: CreateResponse,         // Created (for POST)
    400: ErrorResponse,          // Bad Request
    401: ErrorResponse,          // Unauthorized
    403: ErrorResponse,          // Forbidden
    404: ErrorResponse,          // Not Found
    409: ErrorResponse,          // Conflict
    500: ErrorResponse,          // Server Error
  },
}
```

### Standard Response Patterns by HTTP Method:

**GET (single item)**
```typescript
response: {
  200: ItemResponse,
  401: ErrorResponse,
  404: ErrorResponse,
  500: ErrorResponse,
}
```

**GET (list)**
```typescript
response: {
  200: ListResponse,
  401: ErrorResponse,
  500: ErrorResponse,
}
```

**POST (create)**
```typescript
response: {
  201: CreateResponse,
  400: ErrorResponse,
  401: ErrorResponse,
  403: ErrorResponse,
  409: ErrorResponse,
  500: ErrorResponse,
}
```

**PATCH/PUT (update)**
```typescript
response: {
  200: UpdateResponse,
  400: ErrorResponse,
  401: ErrorResponse,
  404: ErrorResponse,
  500: ErrorResponse,
}
```

**DELETE**
```typescript
response: {
  200: DeleteResponse,
  401: ErrorResponse,
  404: ErrorResponse,
  409: ErrorResponse,
  500: ErrorResponse,
}
```

---

# Elysia - Ergonomic Framework for Humans

> Ergonomic Framework for Humans

Ergonomic Framework for Humans. TypeScript framework supercharged by Bun with End - to - End Type Safety, unified type system and outstanding developer experience

## Table of Contents

### Getting Started

- [At glance - ElysiaJS](https://elysiajs.com/at-glance.md)
- [Quick Start - ElysiaJS](https://elysiajs.com/quick-start.md)
- [Key Concept - ElysiaJS](https://elysiajs.com/key-concept.md)

### Essential

- [Route - ElysiaJS](https://elysiajs.com/essential/route.md)
- [Handler - ElysiaJS](https://elysiajs.com/essential/handler.md)
- [Validation - ElysiaJS](https://elysiajs.com/essential/validation.md)
- [Lifecycle - ElysiaJS](https://elysiajs.com/essential/life-cycle.md)
- [Plugin - ElysiaJS](https://elysiajs.com/essential/plugin.md)
- [Best Practice - ElysiaJS](https://elysiajs.com/essential/best-practice.md)

### Patterns

- [Config - ElysiaJS](https://elysiajs.com/patterns/configuration.md)
- [Reactive Cookie - ElysiaJS](https://elysiajs.com/patterns/cookie.md)
- [Deploy to Production - ElysiaJS](https://elysiajs.com/patterns/deploy.md)
- [Error Handling - ElysiaJS](https://elysiajs.com/patterns/error-handling.md)
- [Extends Context - ElysiaJS](https://elysiajs.com/patterns/extends-context.md)
- [Fullstack Dev Server - ElysiaJS](https://elysiajs.com/patterns/fullstack-dev-server.md)
- [Macro - ElysiaJS](https://elysiajs.com/patterns/macro.md)
- [Mount - ElysiaJS](https://elysiajs.com/patterns/mount.md)
- [OpenAPI - ElysiaJS](https://elysiajs.com/patterns/openapi.md)
- [OpenTelemetry Plugin - ElysiaJS](https://elysiajs.com/patterns/opentelemetry.md)
- [Trace - ElysiaJS](https://elysiajs.com/patterns/trace.md)
- [TypeBox (Elysia.t) - ElysiaJS](https://elysiajs.com/patterns/typebox.md)
- [TypeScript - ElysiaJS](https://elysiajs.com/patterns/typescript.md)
- [Testing - ElysiaJS](https://elysiajs.com/patterns/unit-test.md)
- [WebSocket - ElysiaJS](https://elysiajs.com/patterns/websocket.md)

### Eden

- [End-to-End Type Safety - ElysiaJS](https://elysiajs.com/eden/overview.md)
- [Eden Installation - ElysiaJS](https://elysiajs.com/eden/installation.md)
- [Eden Fetch - ElysiaJS](https://elysiajs.com/eden/fetch.md)

#### Eden Treaty

- [Overview - ElysiaJS](https://elysiajs.com/eden/treaty/overview.md)
- [Eden Treaty Parameters - ElysiaJS](https://elysiajs.com/eden/treaty/parameters.md)
- [Eden Treaty Response - ElysiaJS](https://elysiajs.com/eden/treaty/response.md)
- [Eden Treaty Web Socket - ElysiaJS](https://elysiajs.com/eden/treaty/websocket.md)
- [Eden Treaty Config - ElysiaJS](https://elysiajs.com/eden/treaty/config.md)
- [Eden Treaty Unit Test - ElysiaJS](https://elysiajs.com/eden/treaty/unit-test.md)
- [Eden Treaty Legacy - ElysiaJS](https://elysiajs.com/eden/treaty/legacy.md)

### Plugins

- [Plugin Overview - ElysiaJS](https://elysiajs.com/plugins/overview.md)
- [Bearer Plugin - ElysiaJS](https://elysiajs.com/plugins/bearer.md)
- [CORS Plugin - ElysiaJS](https://elysiajs.com/plugins/cors.md)
- [Cron Plugin - ElysiaJS](https://elysiajs.com/plugins/cron.md)
- [Apollo GraphQL Plugin - ElysiaJS](https://elysiajs.com/plugins/graphql-apollo.md)
- [GraphQL Yoga Plugin - ElysiaJS](https://elysiajs.com/plugins/graphql-yoga.md)
- [HTML Plugin - ElysiaJS](https://elysiajs.com/plugins/html.md)
- [JWT Plugin - ElysiaJS](https://elysiajs.com/plugins/jwt.md)
- [OpenAPI Plugin - ElysiaJS](https://elysiajs.com/plugins/openapi.md)
- [OpenTelemetry Plugin - ElysiaJS](https://elysiajs.com/plugins/opentelemetry.md)
- [Server Timing Plugin - ElysiaJS](https://elysiajs.com/plugins/server-timing.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inboundemail/inbound](https://github.com/inboundemail/inbound) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
