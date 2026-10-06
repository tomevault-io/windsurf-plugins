---
trigger: always_on
description: > **Package**: `@michijs/michijs`
---

# AGENTS.md — AI Coding Assistant Reference for @michijs/michijs

> **Version**: 2.4.0  
> **Package**: `@michijs/michijs`  
> **License**: MIT  
> **Runtime**: Browser (main), Node.js (JSX-to-string rendering)  
> **Package manager**: Bun (declared in `package.json`)  
> **Dev/build tooling used in this repo**: `@michijs/dev-server` (`michi-server`) — not required for consumers

---

## Architecture

Hexagonal architecture. See [ARQUITECTURE.md](./ARQUITECTURE.md) for full diagram.

```
src/
├── domain/          # Core logic — zero infrastructure dependencies
│   ├── entities/    # Observable, ReactiveValue, ProxiedValue, IdGenerator, garbage-collection, etc.
│   ├── ports/       # Interfaces/contracts (importable by any layer)
│   ├── use-cases/   # hooks/, i18n/, proxyHandlers/
│   ├── utils/       # unproxify, getObservables, bindObservable, dependencyTracker
│   └── typewards/   # isObservable, isReactiveValue
├── shared/          # Utilities, types, type guards — no domain/infra deps
│   ├── types/       # AnyObject, PrimitiveType, KebabCase, DeepReadonly, etc.
│   ├── typewards/   # hasToJSON, isProxiedValue
│   └── utils/       # formatToKebabCase, clone/*, throttle, debounce, pick, omit, etc.
├── infrastructure/
│   ├── dom/         # Browser adapter: custom elements, JSX, rendering, styles, routing, storage, URL
│   ├── platform/    # DOM-agnostic JS APIs: fetch, EventDispatcher, constants
│   ├── node/        # Node.js adapter: JSX runtime + NodeElementFactory (renders JSX to string)
│   └── plugin/      # esbuild plugin: compile-time JSX transform (michiJSXPlugin, transformSource, PluginElementFactory)
├── index.ts         # Browser entry (re-exports domain + dom + shared + platform)
├── index.node.ts    # Node entry (re-exports domain + node adapter + shared + platform)
├── jsx-runtime.tsx  # JSX runtime re-export
└── jsx-dev-runtime.tsx
```

### Subpath Imports

Defined in `package.json` `"imports"` field (Node.js subpath imports):

| Import | Resolves to |
|--------|-------------|
| `#ports` | `./src/domain/ports/index.ts` |
| `#domain` | `./src/domain/index.ts` |
| `#shared` | `./src/shared/index.ts` |

### Import Rules

| Layer | Can import from | CANNOT import from |
|-------|----------------|-------------------|
| `shared/` | `#ports` (type-only only) | `#domain` (values), infrastructure |
| `domain/` | `#shared`, `#ports`, relative within domain | infrastructure |
| `infrastructure/` | `#domain`, `#ports`, `#shared` | relative paths crossing layers |

Within the same layer, use **relative imports**. Cross-layer imports use **aliases**.

---

## Reactive System

### Two Observable Modes

The reactive system has two modes, controlled by the `useProxied` parameter:

1. **Lightweight (default)**: `CallableReactiveValuePort<T>` — similar to TC39 signals. Call it to update, subscribe to changes.
2. **Deep-proxy**: `ObservableProxyPort<T>` — intercepts property access/mutations on complex objects (arrays, maps, sets, dates, plain objects).

```ts
// Default — lightweight reactive value
const count = useObserve(0);           // CallableReactiveValuePort<number>
count(5);                               // update
count.subscribe(v => console.log(v));   // subscribe

// Deep-proxy mode
const state = useObserve({ a: 1 }, true);  // ObservableProxyPort<{a:number}>
state.a = 2;                                // triggers subscribers
```

### Key Types (from `@ports`)

| Type | Description |
|------|-------------|
| `ObservablePort<T>` | Base observable interface (has `subscribe`, `notify`) |
| `ReactiveValuePort<T>` | Lightweight reactive value |
| `CallableReactiveValuePort<T>` | Callable reactive value — `obs(newValue)` to update |
| `ObservableProxyPort<T>` | Deep-proxy observable (callable, dispatches to Date/Array/Map/Set/Object/Primitive variants) |
| `ProxiedValuePort<T>` | Port for the proxied value entity |
| `ProxyHandlerPort<T>` | Proxy handler interface |
| `ParentSubscription` | Parent subscription for nested proxied observables |
| `UnproxifyPort<T>` | Type utility to unwrap proxied types |
| `ObservableOrConst<T>` | `T | ObservableProxyPort<T> | CallableReactiveValuePort<T>` |
| `Subscription` | Subscription callback type |
| `TargetPort` | Target for observable notifications |

### Entities (from `@domain`)

| Entity | Description |
|--------|-------------|
| `Observable` | Base observable class |
| `ReactiveValue` | Lightweight reactive value class |
| `ProxiedValue` | Deep-proxy observable class |
| `ProxiedArray` | Proxied array with reactive mutations |
| `ReactiveArray` | Reactive array |
| `Callable` | Makes a class callable |
| `CallableObservable` | Callable observable |
| `ObservableWithValue` | Observable that stores a value |
| `ObservableFromEventListener` | Observable from DOM event listeners |
| `IdGenerator` | Sequential ID generator |
| `MappedIdGenerator` | Mapped ID generator (key-based) |
| `GarbageCollectableObject` | WeakRef-based garbage collection |

---

## Hooks

All hooks are in `domain/use-cases/hooks/`.

### `useObserve(item, useProxied?)`

Creates an observable for a value.

- `useProxied: false` (default) → `CallableReactiveValuePort<T>`
- `useProxied: true` → `ObservableProxyPort<T>`

### `useComputedObserve(callback, options?)`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [michijs/michijs](https://github.com/michijs/michijs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
