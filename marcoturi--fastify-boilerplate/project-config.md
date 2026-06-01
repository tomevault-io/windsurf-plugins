---
trigger: always_on
description: > Instructions for AI coding assistants (Cursor, Claude Code, GitHub Copilot, etc.)
---

# AGENTS.md

> Instructions for AI coding assistants (Cursor, Claude Code, GitHub Copilot, etc.)

## Project overview

A production-ready Fastify 5 boilerplate using Clean Architecture, CQRS, DDD, and functional programming.
TypeScript strict mode, ESM-only, Node >= 24 (native TS execution, no build step).

## Quick reference

| What | Where |
|---|---|
| Package manager | `pnpm` (never npm or yarn) |
| Linter + formatter | Biome (never ESLint or Prettier) |
| Validation after changes | `pnpm check` (runs `biome check && tsc --noEmit`) |
| Auto-fix formatting | `pnpm format` |
| Unit tests | `pnpm test:unit` (node:test) |
| E2E tests | `pnpm test:e2e` (Cucumber + Gherkin) |
| Architecture validation | `pnpm deps:validate` (dependency-cruiser) |
| DB migrations | `pnpm db:migrate` (DBMate) |

Always run `pnpm check` after making changes. If formatting fails, run `pnpm format` first, then `pnpm check`.

## Architecture

### Layer boundaries (CRITICAL)

The dependency flow is strictly **inward**: `Route → Handler → Domain → Repository`.

```
src/
├── instrumentation.ts ← OpenTelemetry setup (loaded via --import before the app)
├── modules/         ← Feature code (vertical slices)
│   └── <feature>/
│       ├── commands/ ← State-changing operations
│       ├── queries/  ← Data-retrieval operations (idempotent)
│       ├── database/ ← Repository port (interface) + adapter (implementation)
│       ├── domain/   ← Business logic, types, errors
│       └── dtos/     ← Shared response schemas
├── server/          ← Fastify setup, plugins, DI wiring
└── shared/          ← Cross-cutting: CQRS, DB, exceptions, utils
```

**Never** import from `src/shared/db/` or `database/` inside handler files.
Handlers interact with data exclusively through repository ports (interfaces).
SQL belongs in repository files only.

### Module independence

Modules should avoid importing directly from other modules. Cross-module communication
uses the CQRS buses:
- Commands/queries for request-response
- Events for fire-and-forget notifications

## CQRS pattern

This project uses three buses: `CommandBus`, `QueryBus`, and `EventBus`.

### Action creators

Actions are created via `actionCreatorFactory` with a module prefix.
The `Result` type parameter is a phantom type — it exists only at the type level
to enable type-safe `execute()` calls:

```typescript
// In the module's index.ts
export const userActionCreator = actionCreatorFactory('user');

// In the handler file — embed both Payload and Result types
export type CreateUserResult = string;
export const createUserCommand = userActionCreator<CreateUserRequestDto, CreateUserResult>('create');
```

### Handler pattern

Every handler follows this structure:

```typescript
export default function makeHandler({ commandBus, repository, ...deps }: Dependencies) {
  return {
    async handler({ payload }: HandlerAction<typeof myCommand>): Promise<MyResult> {
      // business logic here
    },
    init() {
      commandBus.register(myCommand.type, this.handler);
    },
  };
}
```

Key rules:
- Handler parameters use `HandlerAction<typeof creator>` (strips the phantom Result type for register() compatibility)
- The result type (e.g. `CreateUserResult`) is the **unwrapped** value, not `Promise<X>` — the `Promise` wrapper comes from `execute()`
- Events use `userActionCreator<PayloadType>('event-name')` without a Result generic (events return void)
- Handlers are auto-loaded and wired via Awilix DI — the `init()` method is called automatically

### Calling from routes/resolvers

```typescript
// Return type is inferred from the action's phantom Result — no manual generic needed
const id = await fastify.commandBus.execute(createUserCommand(req.body));
```

Do **not** pass a manual generic to `execute()`. The type is inferred from the action creator's `Result` parameter.

### Middlewares

Middlewares use an onion-model pipeline (composed via `reduceRight`). They must **never mutate** the action — always spread into a new object:

```typescript
function myMiddleware(action: Action<unknown>, handler: CommandHandler): Promise<unknown> {
  const enrichedAction = { ...action, meta: { ...action.meta, foo: 'bar' } } as Action<unknown>;
  return handler(enrichedAction);
}
```

There are separate types for command/query vs event middlewares:
- `CommandMiddleware` — returns `Promise<unknown>` (used by CommandBus and QueryBus)
- `EventMiddleware` — returns `void` (used by EventBus)

### Bus differences

| Bus | Purpose | Register method | Dispatch method | Handler return |
|---|---|---|---|---|
| `CommandBus` | State-changing mutations | `register(type, handler)` | `execute(action)` → `Promise<R>` (inferred) | `Promise<unknown>` |
| `QueryBus` | Idempotent reads | `register(type, handler)` | `execute(action)` → `Promise<R>` (inferred) | `Promise<unknown>` |
| `EventBus` | Fire-and-forget notifications | `on(type, handler)` | `emit(action)` → `void` | `void` |

Commands and queries share a `createRequestBus` factory in `src/shared/cqrs/request-bus.ts`.
The event bus is a separate implementation in `src/shared/cqrs/event-bus.ts` with a `logger` dependency
for debug-level warnings when events have no subscribers. Note the different API: `on`/`emit` for events

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marcoturi/fastify-boilerplate](https://github.com/marcoturi/fastify-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
