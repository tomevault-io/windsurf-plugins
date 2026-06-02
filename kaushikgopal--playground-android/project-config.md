---
trigger: always_on
description: provides sufficient context for most decisions.
---

# Agent Instructions

## How to Use This Document

This is the **primary entry point** for agents working on this project. It
provides sufficient context for most decisions.

**When you need more detail:**

- USF ViewModels → See `common/usf/USF.md` and related docs in that module
- Navigation patterns → See `common/navigation/NAVIGATION.md`
- Testing patterns and utilities → See @TESTING.md

## USF Documentation (in `common/usf/`)

- **USF.md** — Core patterns for 90% of cases (basic ViewModels, UI integration,
  testing)
- **USF-QUICKSTART.md** — Complete tutorial if new to USF
- **USF-PLUGINS.md** — Plugin composition for complex features (read only when
  needed)
- **USF-TESTING.md** — Comprehensive testing patterns (read only when writing
  tests)
- **USF-TROUBLESHOOTING.md** — Debugging guide (read only when stuck)

**Agent instructions:** Read `common/usf/USF.md` for basic patterns. Read
specialized guides only when the task requires them. Don't read all files
upfront—pull specific guides as needed.

## Project Overview

playground-android is a modern Android development playground showcasing
best-in-class architectural patterns and practices. It serves as a reference
implementation for building maintainable, testable Android applications using
contemporary tooling and techniques.

**Technology stack**: Kotlin-first with Jetpack Compose, Clean Architecture,
Unidirectional State Flow (USF) ViewModels, and compile-time DI via Metro
(compiler plugin + graph extensions). Modular by design: features ship as
isolated modules, domain logic stays UI-agnostic, and shared utilities live in
`:common`.

## Important Commands

- `make` — Default build target (runs full build including tests)
- `make build-debug` — Assemble debug build without lint
- `make clean` — Remove build outputs and caches
- `make tests` — Execute all unit tests
- `make lint` — Run Android/Kotlin lint checks
- `make ktfmt` — Format staged Kotlin files
- `make ktfmt-all` — Format every Kotlin file

## Core Principles

### 1. Architecture First

- Clean separation of concerns with feature modules, domain logic, and shared
  utilities
- Testability as a first-class concern; every component should be easily
  testable in isolation
- Type-safe dependency injection with compile-time validation

### 2. Modern Development Practices

- Kotlin-first with coroutines for concurrency
- Jetpack Compose for declarative UI with minimal boilerplate
- Unidirectional data flow (USF) for predictable state management
- Immutable data structures to prevent bugs

### 3. Performance & Responsiveness

- Never block the UI; all long-running work happens on background threads
- Efficient state updates that minimize recomposition
- Proper lifecycle management to prevent leaks

### 4. Developer Experience

- Clear patterns that scale from simple to complex features
- Comprehensive documentation with practical examples
- Debugging-friendly architecture with observable state flows

# Architecture

## Module Layout

- `:app` — Entry point; wires dependency graph, root composables, navigation
  display
- `:features:*` — Independent feature modules that expose navigation routes, DI
  components, and USF-driven ViewModels for their screens
- `:domain:*` — Shared business logic and domain models; no Android UI
  dependencies
- `:common:*` — Cross-cutting utilities (logging, networking, USF core, UI
  components, lint rules) that features and domains depend on
- `:build-logic` — Gradle convention plugins that enforce consistent
  configuration across modules

## Unidirectional State Flow (USF)

- Every screen uses `UsfViewModel<Event, UiState, Effect>` with immutable state
  and one-off effects
- Events process on the main thread (`Dispatchers.Main.immediate`); keep
  handlers fast and use `offload { }` for heavy work
- Events flow through `process` inside `ResultScope`; state updates via
  `updateState { … }`, side effects via `emitEffect`
- Model callbacks as part of state to keep composables dumb:
  - Use `inputEventCallback` for simple events:
    `onAction = inputEventCallback(Event.Action)`
  - Use `inputEventCallback` for parameterized callbacks:
    `onTextChanged = inputEventCallback(Event::TextChanged)`
  - These helpers ensure **reference stability** for Compose, preventing
    unnecessary recompositions
  - Callbacks are created once and preserved via `.copy()`, ensuring stable
    references across state updates
- Debug builds enable StrictMode (see `StrictModeInitializer`) to flag blocking
  work on main; fix violations by moving logic into `offload { }` or background
  coroutines
- Naming convention: abbreviate long feature names (e.g., `ULEvent`,
  `ULUiState`, `ULEffect` for `UserListScreen`)

### Plugins for Complex Screens

- Compose ViewModels from reusable `UsfPluginInterface` implementations when
  multiple concerns (search, pagination, filtering) must coexist
- Register plugins via
  `register(plugin = …, mapEvent = …, applyState = …, mapEffect = …, transformEffect = …)`
  to translate between ViewModel and plugin types
- Plugins can hold internal state and react to subscription lifecycle
  (`onSubscribed`, `onUnsubscribed`) for resource management
- Example use cases: search plugin with debouncing, pagination plugin for lists,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaushikgopal/playground-android](https://github.com/kaushikgopal/playground-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
