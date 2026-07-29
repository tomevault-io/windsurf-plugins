---
trigger: always_on
description: These instructions describe project-specific conventions. **Do not flag the patterns below as defects** — they are intentional. Use this file to calibrate review feedback before commenting.
---

# Copilot review instructions for parksapi

These instructions describe project-specific conventions. **Do not flag the patterns below as defects** — they are intentional. Use this file to calibrate review feedback before commenting.

> Authoritative source for conventions: [`CLAUDE.md`](../CLAUDE.md). This file is a Copilot-focused condensation; if there is a conflict, `CLAUDE.md` wins.

## Project overview

parksapi is a TypeScript ESM library that fetches real-time theme park data (wait times, schedules, entities) from 75+ park implementations. All park integrations live under `src/parks/<park>/`. The core abstraction is the `Destination` base class with a Template Method pattern: subclasses implement `_init()`, `buildEntityList()`, `buildLiveData()`, `buildSchedules()`, plus override `getDestinations()` to declare the resort. The other public `get*()` methods (`getEntities`, `getLiveData`, `getSchedules`) must **not** be overridden.

Node 24+, npm 11+, ES2022 modules, decorators enabled.

## Decorator-based design

Five decorators carry most of the framework:

| Decorator | Purpose |
|---|---|
| `@destinationController({category})` | Auto-registers the destination and applies `@config` to the class. **Never apply `@config` to a class directly when using this decorator.** |
| `@config` | Property-level config injection. Values resolve from instance config → `{CLASSNAME}_{PROP}` env var → `{PREFIX}_{PROP}` env var → property default. |
| `@cache({ttlSeconds, key?, callback?, cacheVersion?})` | SQLite-backed memoisation with TTL. |
| `@http({cacheSeconds?, retries?, ...})` | Queue-based HTTP request wrapper. |
| `@inject({eventName, hostname?, tags?, priority?})` | Sift.js (MongoDB-like) event filter for cross-cutting concerns (auth, headers, error handling). |

## Patterns the project uses (do **not** flag these)

### 1. Empty-string config defaults are intentional

```ts
@config apiBase: string = '';
@config token: string = '';
@config email: string = '';
```

`CLAUDE.md` requires that **no URLs, keys, tokens, or credentials ever appear hardcoded in source**. Empty-string defaults force configuration via `.env` and are the documented convention. **Do not suggest providing fallback URLs, default keys, or upfront validation that throws on empty strings.** The HTTP layer's URL-construction error is the documented failure signal when env vars are missing.

**App version strings are an exception** — they are not secrets and may carry sensible defaults (still overridable via env):

```ts
@config appVersion: string = '4.1.10';   // OK — versions can default
```

It is fine — and preferred — to throw a clear error when a credential is needed *for a specific operation* (e.g., `if (!this.email || !this.password) throw new Error('… requires …_EMAIL and …_PASSWORD …')` inside the auth method that needs it). What you should not suggest is gating the whole class on non-empty `apiBase` upfront.

### 2. `this` in `@inject` callbacks does not need typing

```ts
@inject({
  eventName: 'httpRequest',
  hostname: function () { return hostnameFromUrl(this.apiBase); },
})
```

The `function () { ... this.apiBase ... }` form without an explicit `this: ClassName` parameter is used by 10+ destinations and `tsc --noEmit` passes. Do not suggest adding `this: FooDestination` or rewriting as an arrow function (the latter would change `this` binding and break the pattern).

### 3. Entity IDs are always strings

Even when the upstream API provides numbers, the framework requires string IDs. `String(numericId)` is the standard idiom; do not flag this as redundant.

### 4. Numeric validation uses `Number.isFinite`, not `isNaN`

```ts
const n = Number(raw);
if (Number.isFinite(n)) { ... }
```

`isNaN("")` returns `false` (truthy "is not NaN"), which is dangerous for waitTime fields coming from APIs that may return empty strings. Use `Number.isFinite(Number(x))` or `parseInt(x, 10)` paired with `Number.isFinite`. **Do not suggest replacing `Number.isFinite` with `isNaN`.**

### 5. `as any` casts on framework boundaries are tolerated

The `HTTPObj`/`Entity`/`LiveData`/`EntitySchedule` types from `@themeparks/typelib` are intentionally strict to enforce shape at the public API boundary. Park implementations frequently use `as any as HTTPObj` when constructing request objects from minimal field sets, and `as Entity` / `as EntitySchedule` when assembling output. **Do not suggest replacing these casts with full-shape literal construction** unless the cast is hiding an actual type mismatch.

### 6. Throwing inside cached/wrapped methods is fine

Methods decorated with `@cache`, `@reusable`, or that go through `CacheLib.wrap` are expected to throw on auth failures, missing config, or upstream errors. The framework handles propagation. **Do not suggest wrapping these in try/catch unless there is a specific recovery path** that makes sense (e.g. `EMAIL_NOT_FOUND` → fall back to sign-up).

### 7. Most public-method overrides are forbidden — but `getDestinations()` is the supported exception


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ThemeParks/parksapi](https://github.com/ThemeParks/parksapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
