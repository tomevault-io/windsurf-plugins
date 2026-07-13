---
trigger: always_on
description: Lightweight, decorator-driven HTTP framework for TypeScript. Inspired by Angular/NestJS architectural ideas - decorator-driven DI, controllers, modules - Koa's onion-model middleware, and Hono's multi-runtime, lightweight philosophy. Built from scratch on modern standards.
---

# MiiaJS

Lightweight, decorator-driven HTTP framework for TypeScript. Inspired by Angular/NestJS architectural ideas - decorator-driven DI, controllers, modules - Koa's onion-model middleware, and Hono's multi-runtime, lightweight philosophy. Built from scratch on modern standards.

## Philosophy

- **Web Standards first** - Request/Response API, no Express/Fastify lock-in
- **TC39 native decorators** - no reflect-metadata, no experimental flags
- **ESM-only** - ES2025 target, `nodenext` module resolution, `verbatimModuleSyntax`
- **Runtime-agnostic** - Bun, Deno natively; Node.js/uWebSockets via server packages
- **Minimal abstractions** - Koa-style middleware replaces interceptors/pipes/filters. One concept, full power
- **Flat DI** - per-app container, no module scoping by default. Simple beats "correct"
- **Lightweight schemas** - use Drizzle/Papr/Mongoose schemas directly, no decorator-based ORM layer

## Monorepo structure

```
packages/
  core/          - DI, decorators, router, middleware, exceptions, response, cors, logger
  config/        - ConfigModule, ConfigService, validated env via Zod
  serve-static/  - Static file serving (Range, ETag, charset, SPA fallback, dotfile guard)
  node-server/   - Node.js HTTP server (optimized + native modes)
  uws-server/    - uWebSockets.js HTTP server (optimized + native modes)
  auth/          - Strategy-based auth, JWT (jose), Local
  rate-limit/    - Fixed-window rate limiting: rateLimit middleware, RateLimitGuard, @RateLimit/@SkipRateLimit, pluggable stores
  drizzle/       - Drizzle ORM integration (postgres/mysql/sqlite)
  papr/          - MongoDB integration via Papr
  mongoose/      - MongoDB integration via Mongoose
  swagger/       - OpenAPI 3.1 spec generation, Swagger UI serving
  messaging/         - Decorator-driven message bus (event-bus pattern), retry, DLQ, idempotency, named buses, W3C tracing
  messaging-redis/   - Redis Streams transport for messaging (consumer groups, ZSET retry)
  testing/       - TestApp harness for integration tests
  cli/           - Dev CLI: dev, build, start, check, new commands
examples/
  drizzle-app/   - CRUD with Drizzle + PostgreSQL
  papr-app/      - CRUD with Papr + MongoDB
  mongoose-app/  - CRUD with Mongoose + MongoDB
  full-app/      - Full stack example using auth, drizzle, jwt, swagger, serve-static
  messaging-app/ - Event-driven orders flow with @miiajs/messaging + Redis Streams transport
  uws-app/       - Minimal CRUD on @miiajs/uws-server (Node-only)
apps/
  website/       - Documentation site (Nuxt 4)
```

Benchmarks live in a separate repo: [github.com/miiajs/benchmarks](https://github.com/miiajs/benchmarks).

## Tooling

- **Package manager:** Bun (`bun install`, `bun add`)
- **Test runner:** Bun (`bun test`), tests import from `bun:test`
- **Build:** `tsc --build` with composite project references (`tsconfig.build.json`)
- **Formatter:** Biome - single quotes, trailing commas, semicolons as needed (`biome.json`)
- **Git hooks:** Lefthook - pre-commit auto-formats staged files (`lefthook.yml`)
- **Root scripts:** `build`, `build:watch`, `clean`, `test`, `typecheck`, `format`, `format:check`

## Key patterns

### DI: per-app container with inject()

Each `Miia` instance owns its own `Container`. Use `inject(Token)` in a field initializer to resolve dependencies from the active container during class instantiation.

Three scopes: `singleton` (default), `transient` (new instance per resolve), `request` (per HTTP request, cleared after response).

Lifecycle hooks via duck-typing (no interface required): `onInit(): Promise<void>` called during `container.initAll()`, `onDestroy(): Promise<void>` called during `container.destroyAll()`.

`runInContainerContext(container, fn)` executes a function with a specific container as the active context - used internally by module loader and available for advanced use cases.

### Decorators: Symbol.metadata

`@Injectable`, `@Controller`, `@Module` store metadata via TC39 `Symbol.metadata` (polyfilled in `@miiajs/core`). No WeakMaps, no pending drain.

Metadata helpers: `getMeta()`, `setMeta()`, `pushMeta()`, `addToMapMeta()`, `setInMapMeta()`.

External packages use decorator creators for custom decorators:
- `createClassDecorator()` - class-level (e.g. `@ApiTag`)
- `createMethodDecorator()` - method-level (e.g. `@ApiOperation`)
- `createFieldDecorator()` - field-level
- `createDecorator()` - dual class/method (e.g. `@SkipGuard`, `@ApiSecurity`)

### Module system: @Module and dynamic modules

`@Module({ imports, controllers, providers, prefix })` groups related functionality. Modules can import other modules (recursive, circular-safe). `prefix` composes with controller prefixes via `joinPaths()`.

Dynamic module pattern for runtime configuration:
```ts
DrizzleModule.configure((resolve) => {
  const config = resolve(ConfigService)
  return { dialect: 'postgres', connection: { url: config.getOrThrow('DATABASE_URL') } }
})
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [miiajs/miia](https://github.com/miiajs/miia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
