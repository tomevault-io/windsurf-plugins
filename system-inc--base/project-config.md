---
trigger: always_on
description: Modular framework for building **Cloudflare Worker**–based applications. This file orients you to the whole monorepo; each package and each major subsystem has its own `CLAUDE.md` that goes deeper. Read this first, then drill into the package you're working in.
---

# Base

Modular framework for building **Cloudflare Worker**–based applications. This file orients you to the whole monorepo; each package and each major subsystem has its own `CLAUDE.md` that goes deeper. Read this first, then drill into the package you're working in.

## What this is

`base` lets you build a Cloudflare Worker as a set of declarative **modules**. You define services, GraphQL resolvers, queue processors, scheduled tasks, RPC services, and ORM entities using **decorators**; the framework wires them together through a **dependency-injection container** and dispatches platform events (HTTP requests, queue messages, cron triggers, WebSocket events) to your code. Via a platform-delegate abstraction the same app model also targets **Node** — a first-class deployment target for complete Node-based workers (and what tests run on), minus Cloudflare-specific bindings.

## How a base app fits together

**Build model — declare, then register (registration is explicit, never magic).** You write a class, decorate it with what it _is_, list it once in `services`, and add the module to a worker's `BaseSettings.modules`:

```
decorated class                          module                          worker
@OrmTable ──────────────────────►  BaseModule.create({             BaseSettings.modules: [
@HttpService / @GqlResolver /         settings: {              ──►      AccountModule,
@RpcService / @WorkerQueueProcessor /   orm.entities,                   BillingModule, …
@ScheduledExecutable /          ──►     services, … } })            ]
@EventBusListener / @Injectable
```

The decorator `mark`s the class in a `DecoratorRegistry`; at boot the manifest sorts each `services` class into its dispatch surface(s) by those marks — the decorator is the single statement of a class's role, and `services` states only that it exists here. A listed class with no recognized decorator is a boot error (it could do nothing, so listing it is a mistake). Entities register under `orm.entities`; configuration-carrying registrations (webSocket delegates, middleware, GraphQL directives, the access-control provider) keep their own settings slots.

**Runtime lifecycle:**

```
BaseWorker.create(settings)      exports fetch / queue / scheduled (the Cloudflare Worker shape)
  └─ first event ─► Base.initialize()              (idempotent)
        • build the @worker DI container
        • flatten all modules (+ their `uses` deps) into a BaseAppManifest
        • validate every registered class carries its decorator
        • bind GraphQL / RPC / WebSocket / HTTP routes
  └─ each event ─► a fresh scoped child container   (@request | @queue | @scheduled | @websocket)
        • the dispatcher resolves the handler from DI
        • runs middleware, deserializes + validates the input
        • invokes the handler ─► Response
        • deferred work (queue drain, etc.) runs in executionContext.waitUntil, then the container disposes
```

DI is a scope hierarchy `@global → @worker → {@request | @queue | @scheduled | @websocket}`; classes opt into a lifetime with `@Singleton` / `@WorkerScoped()` / `@ContainerScoped()`. Validation runs the same way (`validate()`) across HTTP, RPC, and GraphQL. A platform delegate makes the identical flow run on Cloudflare or Node.

**Read in this order for the full mental model:**

1. [foundation `CLAUDE.md`](./packages/foundation/CLAUDE.md) — the design patterns + subsystem map.
2. [`module/`](./packages/foundation/source/module/CLAUDE.md) — how you register everything (`ModuleSettings`).
3. [`base/`](./packages/foundation/source/base/CLAUDE.md) — boot + dispatch lifecycle.
4. [`configuration/`](./packages/foundation/source/configuration/CLAUDE.md) — `BaseConfiguration`, the manifest, decorator validation.
5. [`dependency-injection/`](./packages/foundation/source/dependency-injection/CLAUDE.md) — scopes, inject decorators, typed tokens.
6. Then the subsystem you're touching — [orm](./packages/foundation/source/orm/CLAUDE.md), [graphql](./packages/foundation/source/graphql/CLAUDE.md), [rpc](./packages/foundation/source/rpc/CLAUDE.md), [router/http](./packages/foundation/source/router/CLAUDE.md), [queue](./packages/foundation/source/queue/CLAUDE.md), [scheduled](./packages/foundation/source/scheduled/CLAUDE.md), [web-socket](./packages/foundation/source/web-socket/CLAUDE.md), [access-control](./packages/foundation/source/access-control/CLAUDE.md), …

## Repository layout

npm **workspaces** monorepo (`packages/*` + `examples`).

| Package                                                  | npm name                      | Role                                                                                               |
| -------------------------------------------------------- | ----------------------------- | -------------------------------------------------------------------------------------------------- |
| [`packages/lint`](./packages/lint/CLAUDE.md)             | `@system-inc/base-lint`       | Shared ESLint config + custom rules (ESM, no build step)                                           |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [system-inc/base](https://github.com/system-inc/base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
