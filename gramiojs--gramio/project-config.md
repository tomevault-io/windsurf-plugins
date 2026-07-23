---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is this?

GramIO — a TypeScript Telegram Bot API framework. Multi-runtime (Node.js, Bun, Deno), type-safe, extensible via plugins and hooks. The npm package is `gramio`, JSR package is `@gramio/core`.

## Commands

```bash
bun test                # Run all tests (Bun's built-in test runner)
bun test tests/foo.test.ts  # Run a single test file
bun run type            # Type-check (tsc --noEmit)
bun run lint            # Lint with Biome
bun run lint:fix        # Auto-fix lint issues
bunx pkgroll            # Build (dual CJS/ESM via pkgroll)
```

## Architecture

### Core classes (all in `src/`)

- **Bot** (`bot.ts`, ~1400 lines) — Main class. Three generics: `Bot<Errors, Derives, Macros>`. Holds the API proxy (`bot.api.*`), update handlers (`.on()`, `.command()`, `.hears()`, `.callbackQuery()`, `.reaction()`, `.inlineQuery()`, `.startParameter()`), plugin system (`.extend()`), derive system (`.derive()`, `.decorate()`), macro system (`.macro()`), hooks, and lifecycle (`.start()`, `.stop()`). `.on()` has 5 overloads: filter-only type-narrowing, filter-only boolean, event+type-narrowing filter, event+boolean filter, and event-only (no filter). Shorthand methods (command, hears, callbackQuery, etc.) accept an optional `options` parameter for macro activation and preHandler arrays.
- **Plugin** (`plugin.ts`) — Same handler/hook/derive/macro API as Bot for composability. Three generics: `Plugin<Errors, Derives, Macros>`. Plugins merge their types (including macros) into the bot instance. Has dependency management between plugins. `.on()` mirrors Bot's 5 overloads.
- **Filters** (`filters.ts`) — Standalone predicate functions for narrowing context types in `.on()` handlers. Exports `Filter` type and `filters` object with built-in filters (attachment, text, caption, chat, from, regex, etc.) and composition (`and`, `or`, `not`). Filters use `any` as input type with intersection-based narrowing (`ContextType<T> & Narrowing`) so the handler preserves the full context type while adding type narrowing.
- **Composer** (`composer.ts`) — Middleware chain built on `@gramio/composer`. Sequential execution with `next()`. Both `.on(updateName, handler)` and `.use(handler)`. Custom methods (command, hears, callbackQuery, etc.) are defined here via `createComposer({ methods })` and wired with `buildFromOptions()` for macro support.
- **Updates** (`updates.ts`) — Long-polling manager. `.handleUpdate()` dispatches a single update through the middleware chain.
- **UpdateQueue** (`queue.ts`) — Concurrent update processing queue with graceful shutdown support.
- **Webhook handler** (`webhook/`) — `webhookHandler()` function with adapters for Fastify, Elysia, Hono, Express, Koa, Bun.serve, Cloudflare Workers, std/http, node:http.

### Hook system (6 hooks)

`preRequest` (before API calls, mutable) → `onResponse` / `onResponseError` (after API calls) → `onError` (handler errors, type-safe with custom error kinds) → `onStart` / `onStop` (lifecycle).

### Re-exported packages

`src/index.ts` re-exports from `@gramio/contexts`, `@gramio/files`, `@gramio/keyboards`, `@gramio/types`, `@gramio/format`, `@gramio/callback-data`, and `./filters.js`. Changes to context types, file handling, keyboards, or formatting live in those separate packages.

### Key patterns

- **Heavy generics** — Bot class accumulates type parameters (`Errors`, `Derives`, `Macros`) as plugins/derives/errors/macros are chained. Type inference flows through the fluent API.
- **Macro system** — Elysia-inspired declarative handler options. `bot.macro(name, def)` registers macros (from `@gramio/composer`). Shorthand methods accept optional `options` as 3rd argument: `command("start", handler, { throttle: { limit: 3 }, auth: true, preHandler: [guard] })`. `buildFromOptions()` composes the macro chain. Execution order: `options.preHandler` → per-macro `preHandler` → per-macro `derive` → main handler. Macros propagate from Plugin to Bot via `extend()`. `HandlerOptions<TBaseCtx, Macros>` types the options parameter; `DeriveFromOptions<Macros, TOptions>` enriches handler context with macro derives.
- **API proxy** — `bot.api` is a Proxy that maps method names to Telegram Bot API calls. Supports `suppress: true` to return error objects instead of throwing.
- **`debug` logging** — Namespaces: `gramio:api`, `gramio:updates`.
- **Runtime detection** — `IS_BUN` constant switches file upload strategy (Bun has optimized paths).
- **Dual package** — pkgroll builds both CJS and ESM from the same source.

## Testing

### Workflow

After making changes, always run:

1. `bun test` — Run all tests. All must pass.
2. `bun run type` — Type-check. Must have no errors.
3. `bun run lint` — Lint with Biome. Must be clean.

### What we test

Tests here verify **GramIO framework behavior** — handlers, hooks, plugins, middleware, derives, error handling. `@gramio/test` is just a helper that simulates Telegram updates without real HTTP; its own features (UserObject, ChatObject, etc.) are tested in its own repository. Do not write tests whose sole purpose is to verify that `@gramio/test` works.

### Keeping tests up to date


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gramiojs/gramio](https://github.com/gramiojs/gramio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
