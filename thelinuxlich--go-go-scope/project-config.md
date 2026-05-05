---
trigger: always_on
description: go-go-scope is a TypeScript monorepo that provides **structured concurrency** using the Explicit Resource Management proposal (TypeScript 5.2+). It enables developers to write concurrent code with automatic cleanup and cancellation propagation using the `using` and `await using` syntax.
---

# go-go-scope - Agent Documentation

## Project Overview

go-go-scope is a TypeScript monorepo that provides **structured concurrency** using the Explicit Resource Management proposal (TypeScript 5.2+). It enables developers to write concurrent code with automatic cleanup and cancellation propagation using the `using` and `await using` syntax.

### Key Features
- Native Resource Management via `using`/`await using` (ES2022+ Disposable symbols)
- Structured concurrency with automatic parent-child cancellation
- Built-in timeout support with automatic cancellation
- Structured racing where losers are cancelled
- Go-style channels for concurrent communication with `map`, `filter`, `reduce`, `take` operations
- Broadcast channels for pub/sub patterns
- Semaphores for rate limiting
- Circuit breaker pattern for preventing cascading failures
- Retry logic with configurable delays and conditions (exponential backoff, jitter, linear)
- Stream API with 50+ lazy operations (map, filter, flatMap, buffer, debounce, throttle, pairwise, window, etc.)
- `parallel()` for processing arrays with progress tracking and error handling
- Dependency injection via context services
- Polling utilities with start/stop control
- Debounce and throttle rate-limiting utilities
- Select statement for channel operations (Go-style) with timeout
- Lifecycle hooks for task and resource events
- Metrics collection with Prometheus/JSON export
- Resource pools for connection/worker management
- Task profiling for performance analysis
- Deadlock detection
- Structured logging integration
- Checkpoint and idempotency support for fault-tolerant operations
- Graceful shutdown handling with customizable strategies
- Token bucket rate limiting
- Priority channels for task prioritization
- Web Worker pool for CPU-intensive operations
- Distributed job scheduler with cron support, Web UI, and multiple storage backends
- Cache utilities with warming and multi-tier support
- **Property-based testing**: Mathematical properties verified with fast-check
- **Persistence adapters**: Distributed locks and circuit breaker state across Redis, PostgreSQL, MySQL, SQLite, MongoDB, DynamoDB, Deno KV, Cloudflare Durable Objects
- **Framework adapters**: Fastify, Express, NestJS, Hono, Elysia, Koa, Hapi, Next.js, Remix, SvelteKit

### Patterns (via composition)
- **Actor Model**: Message-passing concurrency with channels
- **Health Checks**: Parallel dependency checking with timeouts
- **Pipeline Processing**: Stream-based data transformation pipelines
- **Circuit Breaker**: Fault tolerance with automatic recovery
- **Distributed Scheduling**: Cron-like job scheduling with HA support

---

## Technology Stack

- **Language**: TypeScript 5.9.3+
- **Target**: ES2022 with NodeNext module resolution
- **Required Features**: `Symbol.dispose`, `Symbol.asyncDispose` (ES2022+ or Node.js 24+)
- **Build Tool**: [pkgroll](https://github.com/privatenumber/pkgroll) v2.26.3 - Zero-config TypeScript package bundler
- **Package Manager**: pnpm >= 8.0.0 (workspaces enabled)
- **Versioning**: [changesets](https://github.com/changesets/changesets) for monorepo versioning

### Runtime Requirements
- **Node.js**: >= 24.0.0 (for native `fetch`, `AbortSignal` improvements, and `using` syntax)
- **Bun**: >= 1.2.0 (fully supported)

### Development Tools
- **Linter/Formatter**: [Biome](https://biomejs.dev/) v2.4.4
- **Testing**: [Vitest](https://vitest.dev/) v4.0.18 with globals enabled
- **TypeScript Execution**: [tsx](https://github.com/privatenumber/tsx) v4.21.0

### Runtime Dependencies (Core)
- `debug` ^4.4.3 (for debug logging)

### Peer Dependencies (Optional)
- `@opentelemetry/api` ^1.9.0 (for tracing, in plugin-opentelemetry)
- `ioredis` ^5.9.3 (for Redis persistence)
- `pg` ^8.18.0 (for PostgreSQL persistence)
- `mysql2` ^3.18.0 (for MySQL persistence)
- `sqlite3` ^5.1.7 (for SQLite persistence)
- `fastify` ^5.7.4 + `fastify-plugin` ^5.1.0 (for Fastify adapter)
- Various framework-specific packages for other adapters

---

## Monorepo Structure

```
packages/
├── go-go-scope/                # Core library (main package)
│   ├── src/
│   │   ├── index.ts            # Main exports
│   │   ├── types.ts            # Type definitions
│   │   ├── scope.ts            # Scope class - structured concurrency primitive
│   │   ├── task.ts             # Task class - lazy disposable Promise
│   │   ├── factory.ts          # scope() factory function
│   │   ├── channel.ts          # Go-style channels
│   │   ├── broadcast-channel.ts # Pub/sub broadcast channels
│   │   ├── semaphore.ts        # Rate limiting primitive
│   │   ├── circuit-breaker.ts  # Fault tolerance
│   │   ├── resource-pool.ts    # Managed resource pools
│   │   ├── worker-pool.ts      # Web Worker pool for CPU tasks
│   │   ├── token-bucket.ts     # Token bucket rate limiting
│   │   ├── priority-channel.ts # Priority queue channels
│   │   ├── cache.ts            # Cache utilities
│   │   ├── checkpoint.ts       # Checkpoint/restart support
│   │   ├── idempotency.ts      # Idempotency provider

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thelinuxlich/go-go-scope](https://github.com/thelinuxlich/go-go-scope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
