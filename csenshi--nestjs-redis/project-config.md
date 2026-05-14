---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

## Commands

Nx monorepo using pnpm. Package names for nx targets: `client`, `health-indicator`, `lock`, `socket.io-adapter`, `streams-transporter`, `throttler-storage`.

```bash
# Build/lint/test all affected
pnpm exec nx affected -t lint test build

# Single package
pnpm nx build client
pnpm nx test lock
pnpm nx lint throttler-storage

# Single test file
pnpm nx test <package> --testFile=path/to/file.spec.ts

# Typecheck
pnpm nx typecheck <package>

# Format
pnpm exec nx format:check --all
pnpm exec nx format:write --all

# Release (bump locally, CI publishes after tag push)
pnpm exec nx release minor --skipPublish   # or major / patch / 1.2.3
git push --follow-tags
```

Start Redis before running any tests: `docker compose up redis -d`

## Architecture

Six independently installable NestJS packages under `packages/`, all `@nestjs-redis/*` scoped:

| Package               | Purpose                                                                                                |
| --------------------- | ------------------------------------------------------------------------------------------------------ |
| `client`              | Core. DI-managed Redis connections (Client/Cluster/Sentinel). All other packages depend on this.       |
| `health-indicator`    | Terminus health check integration.                                                                     |
| `lock`                | Distributed locking via `@redis-kit/lock` (`RedlockModule`, `RedlockService`, `@Redlock()` decorator). |
| `socket.io-adapter`   | Redis adapter for Socket.IO horizontal scaling.                                                        |
| `streams-transporter` | Redis Streams microservices transport with consumer groups.                                            |
| `throttler-storage`   | `ThrottlerStorage` impl using Lua scripting for atomic rate limiting.                                  |

### Module pattern

All packages use `ConfigurableModuleBuilder` from `@nestjs/common`:

```typescript
// module-definition.ts sets .setClassMethodName('forRoot') and factory name
export const { ConfigurableModuleClass, MODULE_OPTIONS_TOKEN } =
  new ConfigurableModuleBuilder<RedisModuleOptions>({
    moduleName: 'RedisClient',
  })
    .setClassMethodName('forRoot')
    .setFactoryMethodName('createRedisOptions' as keyof RedisOptionsFactory)
    .build();
```

The `client` module exports tokens via `RedisToken(connectionName?)`:

- No name → token `'REDIS_CLIENT'`
- Named `'cache'` → token `'REDIS_CLIENT_CACHE'` (uppercased)

Use `@InjectRedis(connectionName?)` or `Inject(RedisToken(connectionName?))` to inject clients.

`RedisModuleOptions` = connection config only (`type`, `options`). `RedisModuleForRootOptions` adds `isGlobal` and `connectionName`. The `useFactory` in `forRootAsync` returns `RedisModuleOptions` — **not** `RedisModuleForRootOptions`.

### Testing

- Tests: `*.spec.ts` (unit) and `*.int.spec.ts` (integration) — Redis must be running for all tests
- Runner: Jest with SWC compilation
- Start Redis: `docker compose up redis -d`

## Code Style & Conventions

- **redis package**: node-redis v5 (`redis ^5.0.0`), **not** ioredis. Types: `RedisClientType`, `RedisClusterType`, `RedisSentinelType`.
- **TypeScript**: strict mode, `target: ES2022`, `module: nodenext`, `moduleResolution: nodenext`, `customConditions: ["development"]`, `noUnusedLocals`, `noImplicitReturns`.
- **No barrel re-exports inside lib/**: `index.ts` at `src/` level only. Internal imports use direct paths.
- **Client lifecycle**: `RedisModule` connects on startup, disconnects on `onApplicationShutdown`. Other services (e.g., `RedisThrottlerStorage`) do **not** manage their client's lifecycle.
- **Lua scripts** in `throttler-storage`: loaded lazily, SHA cached, NOSCRIPT fallback re-runs with raw script.
- **Conventional commits** required; used by Nx release for changelogs.
- **Debug logging**: gated on `process.env['REDIS_MODULE_DEBUG'] === 'true'`; errors always log.

## What Claude Often Gets Wrong

1. **Using ioredis types**: This repo uses `redis` (node-redis v5), not ioredis. Do not use `IORedis`, `Redis` from ioredis, or ioredis-style APIs.

2. **`RedisModuleOptions` vs `RedisModuleForRootOptions`**: `useFactory` in `forRootAsync` must return `RedisModuleOptions` (connection config only — no `isGlobal`/`connectionName`). Those fields are on `RedisModuleForRootOptions` and `RedisModuleAsyncOptions` only.

3. **Token format**: `RedisToken('cache')` produces `'REDIS_CLIENT_CACHE'` (uppercased). Do not construct token strings manually.

4. **`RedlockService` is just `class RedlockService extends Redlock {}`**: It directly extends `Redlock` from `@redis-kit/lock` for DI injection. No wrapper logic — use `Redlock` API directly on the injected service.

5. **No `RedisService` class**: There is no service class in the `client` package. Connections are injected directly via `@InjectRedis()`. Do not create or reference a `RedisService`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CSenshi/nestjs-redis](https://github.com/CSenshi/nestjs-redis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
