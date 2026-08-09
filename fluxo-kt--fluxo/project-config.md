---
trigger: always_on
description: **Fluxo** /ˈfluksu/ — Kotlin Multiplatform state-management on coroutines + `StateFlow`. Combines strict Redux/MVI correctness with MVVM+ ergonomics (suspend lambda intents, side-jobs, time-travel-ready logging). **Pre-1.0 alpha**; public API is unstable but locked per-commit by `binary-compatibility-validator` (BCV): JVM bytecode, Android-main bytecode, JS/Wasm declarations, and KLib ABI. Targets every KMP platform.
---

# Fluxo — Agent Guide

**Fluxo** /ˈfluksu/ — Kotlin Multiplatform state-management on coroutines + `StateFlow`. Combines strict Redux/MVI correctness with MVVM+ ergonomics (suspend lambda intents, side-jobs, time-travel-ready logging). **Pre-1.0 alpha**; public API is unstable but locked per-commit by `binary-compatibility-validator` (BCV): JVM bytecode, Android-main bytecode, JS/Wasm declarations, and KLib ABI. Targets every KMP platform.

## Meta-rule: if you're surprised, alert + amend

If anything in this project surprises you, contradicts the docs, or would have saved you time to know — **mention it in your reply to the user *and*, in the same turn, append a one-line entry (with a symbol or file:line citation) to the "Gotchas" list below**. If a listed gotcha becomes obsolete, remove it. The compounding cost of un-shared discoveries is the single biggest tax on agent work here.

## Composite build (read this first)

`settings.gradle.kts` **conditionally** `includeBuild`s the sibling harness — guard computed *inside* `pluginManagement {}` (it evaluates before the script body, so a top-level val is out of scope there). The Gradle DSL the build relies on (`fkcSetupRaw`, `fkcSetupMultiplatform`, `fkcSetupKotlinApp`, `isRelease()`, `isCI()`) lives in that sibling repo (https://github.com/fluxo-kt/fluxo-kmp-conf). It dogfoods the local sibling **only** when it exists **and** not on CI; otherwise it resolves the **published** `io.github.fluxo-kt.fluxo-kmp-conf` plugin (catalog-pinned), so a fresh single-repo checkout and every CI run build exactly what external consumers get (invariant **I1**). Force either side with `-Pfluxo.dogfood=true|false`. Local default is dogfooding.

`enableFeaturePreview("TYPESAFE_PROJECT_ACCESSORS")` is what makes `projects.fluxoCommon` syntax work in module builds.

## Modules

| Path | Role |
|---|---|
| `:fluxo-common` | Internal annotations only: `@InlineOnly`, `@InternalFluxoApi`, `@ExperimentalFluxoApi`. `compileOnly` upstream. |
| `:fluxo-core` | The state container. `Store`/`StoreSE`/`Container`/`ContainerHost`, `FluxoSettings`, `IntentHandler`/`Reducer`, `Bootstrapper`, `SideJob`, `IntentStrategy` (Fifo/Lifo/Parallel/Direct/ChannelLifo), `SideEffectStrategy` (RECEIVE/CONSUME/SHARE/DISABLE), `GuaranteedEffect`, `StoreFactory`/`StoreDecorator`(`Base`), `repeatOnSubscription`, `closeAndWait`. |
| `:fluxo-data` | `FluxoResult<T>` mixed-state value (NotLoaded/Cached/Loading/Empty/Success/Failure, bit-packed flags). Optional; exposes `fluxo-common` and coroutines as API dependencies. |
| `:benchmarks:jmh` | JVM-only JMH harness comparing Fluxo vs ~14 libs (Ballast, OrbitMVI, MVIKotlin, FlowMVI, Redux, MVICore, …). Single-platform via `fkcSetupKotlinApp`. |

KMP source-set intermediates `commonJvmMain`/`commonJvmTest`, `nonJvmMain`/`nonJvmTest`, `appleTest` are created by `fluxo-kmp-conf`, not stock Kotlin. Test helpers live in package `kt.fluxo.test`; tests proper in `kt.fluxo.tests` — keep the convention.

## Public API entry points

- `container(initial) { … }` → `ContainerS<State>` / `Container<S, SE>` — MVVM+ suspend lambda intents.
- `store(initial, reducer = …)` → strict pure-reducer MVI `Store<I, S>`.
- `store(initial, handler = …)` → discrete intents + MVVM+ `IntentHandler` DSL.
- `CoroutineScope.container/store(...)` extensions inherit caller `coroutineContext`.
- `Container<S, SE>` is a **typealias** for `StoreSE<FluxoIntent<S, SE>, S, SE>` (matters for Java consumers and migration helpers).
- `ContainerS<State>` (no-SE factory) **forcibly sets `sideEffectStrategy=DISABLE`** — these stores cannot post side effects even if you flip the setting later.
- `Store` IS `StateFlow<State>` + `FlowCollector<Intent>` + `CoroutineScope` + `Closeable`. `closeAndWait()` (experimental) drains.
- DSL inside intent/sideJob/bootstrapper (`StoreScope`): `updateState { it + 1 }`, `value = …`, `compareAndSet`, `postSideEffect`, `sideJob(key) { wasRestarted -> … }`, `repeatOnSubscription { … }`, `noOp()`. Guardian rejects `sideJob` blocks that aren't last in the handler.
- All factories accept `settings: FluxoSettings? = null`, `factory: StoreFactory? = null`, `setup: FluxoSettings.() -> Unit = {}`. `setup` runs on a defensive `copy()` of `settings` — original is never mutated.
- `Store` subtypes (`StoreSE`, `StoreScope`, `StoreDecorator`, `StoreFactory`) are `@SubclassOptInRequired(ExperimentalFluxoApi::class)`. For decorators extend `StoreDecoratorBase` (manual delegation, by design — keeps the public surface small).

## Gotchas

Each cites a symbol or file so you can verify in one read.

1. **`Direct` is the default `intentStrategy`.** `FluxoSettings.intentStrategy` initial value and `ParallelIntentStrategy.Companion.DIRECT` (`= ParallelIntentStrategy(start=UNDISPATCHED)`) confirm. Do not reintroduce the old FIFO-default docs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fluxo-kt/fluxo](https://github.com/fluxo-kt/fluxo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
