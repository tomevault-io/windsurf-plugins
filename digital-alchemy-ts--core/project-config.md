---
trigger: always_on
description: `@digital-alchemy/core` is a zero-magic dependency-injection framework for TypeScript. There is no reflection, no decorators, no class hierarchy. Every service is a plain function that receives its dependencies through a single destructured parameter — `TServiceParams`.
---

# CLAUDE.md — @digital-alchemy/core

## 1. What core is

`@digital-alchemy/core` is a zero-magic dependency-injection framework for TypeScript. There is no reflection, no decorators, no class hierarchy. Every service is a plain function that receives its dependencies through a single destructured parameter — `TServiceParams`.

```typescript
export function MyService({ logger, lifecycle, config }: TServiceParams) {
  lifecycle.onReady(() => {
    logger.info("ready");
  });
  return { doThing };
}
```

The framework wires up all declared services at bootstrap time, building a dependency graph from the `libraries` and `services` fields of each `CreateApplication` / `CreateLibrary` call. Once wired, every service receives its full `TServiceParams` synchronously (with async factories awaited).

This repo is the foundation every other `@digital-alchemy` library depends on. Changes to exports, lifecycle semantics, or `TServiceParams` shape ripple downstream into every consumer. Treat public API surface changes as breaking.

---

## 2. Repo orientation

### `src/services/*.mts` — stateful service factories

| File | Owns / responsibility | Read by |
|---|---|---|
| `als.service.mts` | `AsyncLocalStorage` wrapper; `enterWith`, `run`, `getStore`, `getLogData` | Logger (ALS log merging), any code needing per-request context |
| `configuration.service.mts` | Config state map, loader orchestration, proxy for `config.*`, `setConfig`, `mergeConfig`, `validateConfig`, `onUpdate` | `wiring.service.mts` at bootstrap; every service via `config` injection |
| `internal.service.mts` | `InternalDefinition` class (boot state, module maps, lifecycle ref); `InternalUtils` (object path get/set/del, `is`, `relativeDate`, `titleCase`); `safeExec` | All services — injected as `internal` |
| `is.service.mts` | `IsIt` class — type guards and small utilities (`array`, `boolean`, `empty`, `equal`, `function`, `object`, `string`, etc.); exports singleton `is` | Imported directly by `lifecycle.service.mts` and `wiring.service.mts` to break circular deps; everywhere else reached as `internal.utils.is` |
| `lifecycle.service.mts` | `CreateLifecycle` — per-bootstrap lifecycle event registry; priority-sorted callback execution across all seven stages | `wiring.service.mts` owns the lifecycle instance; services attach via `lifecycle.*` injection |
| `logger.service.mts` | `Logger` factory — chalk/stdout formatter, `context(ctx)` builder, level filtering, `addTarget`, `updateShouldLog`, ALS integration, `systemLogger` | Every service via `logger` injection; `internal.boilerplate.logger` |
| `scheduler.service.mts` | `Scheduler` factory — cron, interval, sliding, `setTimeout`, `setInterval`, `sleep`; registers stop callbacks for clean shutdown; returns a builder called with `(context: TContext)` | Every service via `scheduler` injection |
| `wiring.service.mts` | Bootstrap orchestration — `CreateApplication`, `wireService`, `CreateBoilerplate`, `teardown`, SIGINT/SIGTERM handling; owns `LIB_BOILERPLATE` | Entry point for all apps; do not call `wireService` directly |
| `index.mts` | Re-exports all service exports | Everything in `src/` imports from `../index.mts` |

### `src/helpers/*.mts` — pure logic, types, and utilities

**What distinguishes helpers from services:** helpers are side-effect-free modules containing types, utility functions, constants, and small classes. They do not receive `TServiceParams`. Services are factories that receive DI params and return an API object.

| File | Owns |
|---|---|
| `async.mts` | `each`, `eachSeries`, `eachLimit` — async iteration helpers that replace inconsistent `async` library behavior |
| `config-environment-loader.mts` | `ConfigLoaderEnvironment` — reads env vars and CLI switches; search order is `MODULE__KEY` (double-underscore) → `MODULE_KEY` (single-underscore) → `KEY` |
| `config-file-loader.mts` | `configLoaderFile`, `configFilePaths`, `loadConfigFromFile`, `withExtensions` — file-based config loading (JSON, YAML, INI, auto-detect) |
| `config.mts` | All config types (`AnyConfig`, `StringConfig`, `BooleanConfig`, etc.), `ConfigLoaderParams`, `ConfigLoaderReturn`, `findKey`, `iSearchKey`, `loadDotenv`, `parseConfig`, `KnownConfigs` |
| `context.mts` | `TContext` branded string type; `IContextBrand` |
| `cron.mts` | `CronExpression` enum, `TOffset`, `SchedulerCronOptions`, `SchedulerIntervalOptions`, `SchedulerSlidingOptions`, `DigitalAlchemyScheduler`, `SchedulerBuilder` |
| `errors.mts` | `BootstrapException` (wiring-time errors), `InternalError` (runtime errors) — both carry `context`, `cause`, `timestamp` |
| `events.mts` | Global error event name constants (`DIGITAL_ALCHEMY_NODE_GLOBAL_ERROR`, etc.) |
| `extend.mts` | `deepExtend`, `deepCloneArray`, `cloneSpecificValue` — deep merge utilities |
| `index.mts` | Re-exports all helper exports; new public helpers thread through here |
| `lifecycle.mts` | `TLifecycleBase`, `TLifeCycleRegister`, `LIFECYCLE_STAGES` array, `LifecycleStages` type, `LifecycleCallback` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Digital-Alchemy-TS/core](https://github.com/Digital-Alchemy-TS/core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
