---
trigger: always_on
description: A modern, container-based dependency injection system for Swift and SwiftUI. Compile-time safe, lightweight (~1k lines), zero-codegen.
---

# Factory

A modern, container-based dependency injection system for Swift and SwiftUI. Compile-time safe, lightweight (~1k lines), zero-codegen.

This file is the quick orientation. For deep guidance on Factory APIs and idioms, invoke the `factory-dependency-injection` skill in `.claude/skills/factory-dependency-injection/SKILL.md`.

The skill is the in-depth reference. It condenses the DocC catalog and covers: defining factories (standard, constructor injection, `ParameterFactory`, `promised()`), all eight property wrappers (`@Injected`, `@LazyInjected`, `@WeakLazyInjected`, `@DynamicInjected`, `@InjectedObject`, `@InjectedObservable`, `@InjectedContainer`, `@InjectedType`) and when each resolves, scopes and the "factory wins" modifier-ordering rule, contexts and their precedence, custom containers and `AutoRegistering`, SwiftUI/Observation integration, previews, Swift Testing (`.container` trait) and XCTest, cross-module wiring, the global `dependency(\.path)` functions, functional injection, the resolution trace, and a gotchas checklist. Reach for it whenever you touch `Factory<`, `Container.shared`, `extension Container`, `@Injected(\.`, or `import FactoryKit`.

## Repository layout

- `Sources/FactoryKit/FactoryKit/` — the library (the import target is `FactoryKit`, not `Factory`).
  - `Factory.swift` — `Factory<T>` and `ParameterFactory<P, T>`
  - `Containers.swift` — `Container`, `SharedContainer`, `ManagedContainer`, `ContainerManager`, `AutoRegistering`
  - `Injections.swift` — `@Injected`, `@LazyInjected`, `@WeakLazyInjected`, `@DynamicInjected`, `@InjectedObject`, `@InjectedObservable`, `@InjectedContainer`, `@InjectedType`
  - `Scopes.swift` — `.unique`, `.cached`, `.shared`, `.singleton`, `.graph`, custom scopes
  - `Modifiers.swift` — `FactoryModifying`, scope/decorator/context/once/reset/preview
  - `Contexts.swift` — `FactoryContextType` (`.preview`, `.test`, `.debug`, `.simulator`, `.device`, `.arg`, `.args`)
  - `Dependency.swift` — global `dependency(\.keyPath)` / `dependency(\.keyPath, parameter:)`
  - `Resolver.swift` — opt-in typed `Resolving` mode (register/resolve by `T.Type`)
  - `Registrations.swift`, `Key.swift`, `Locking.swift`, `Globals.swift` — internals
- `Sources/FactoryKit/FactoryKit.docc/` — full DocC catalog (Basics, Advanced, Development, Additional). Source of truth for usage docs.
- `Sources/FactoryTesting/ContainerTrait.swift` — Swift Testing `.container` trait + `ContainerTrait<C>` for parallel-safe tests.
- `Tests/` — XCTest and Swift Testing suites (see for canonical usage examples).
- `FactoryDemo/` — sample iOS app exercising the API.
- `Package.swift` — Swift 6 language mode, strict concurrency, products `FactoryKit` and `FactoryTesting`.

## Core mental model

A `Factory<T>` is a transient value type returned from a computed property on a `Container`. Every call to that property builds a fresh `Factory` (cheap, like a SwiftUI `View`). The container — not the `Factory` — owns registrations and scope caches.

```swift
import FactoryKit

extension Container {
    var myService: Factory<MyServiceType> {
        self { MyService() }            // sugar for Factory(self) { ... }
    }
}
```

Resolution:

```swift
let svc = Container.shared.myService()  // service-locator style
let svc = container.myService()          // passed-container style

class VM {
    @Injected(\.myService) var service   // resolved at init
}
```

Mocking is just registering a new closure on the container's factory:

```swift
Container.shared.myService.register { MockService() }
```

## Most common patterns

Constructor injection from the container:

```swift
extension Container {
    var repo: Factory<Repo> { self { Repo(net: self.network()) } }
    var network: Factory<Networking> { self { LiveNetwork() }.singleton }
}
```

Scopes (modifiers on the Factory):

```swift
self { MyService() }           // unique (default — new every call)
self { MyService() }.cached    // cached on the container
self { MyService() }.shared    // weakly cached on the container
self { MyService() }.singleton // global, container-independent
self { MyService() }.graph     // cached for one resolution cycle
self { MyService() }.scope(.session)  // custom scope
```

Parameters require `ParameterFactory` (no property-wrapper form):

```swift
extension Container {
    var paramService: ParameterFactory<Int, ParamService> {
        self { ParamService(value: $0) }
    }
}
let s = Container.shared.paramService(42)
```

SwiftUI:

```swift
@InjectedObject(\.contentViewModel) var vm        // ObservableObject — uses StateObject
@InjectedObservable(\.contentViewModel) var vm    // @Observable (iOS 17+) — uses State
```

Contexts (auto-overrides for environments):

```swift
container.analytics
    .onTest    { MockAnalytics() }   // DEBUG-only
    .onPreview { MockAnalytics() }   // DEBUG-only
    .onDebug   { MockAnalytics() }   // DEBUG-only
    .onArg("mock1") { MockServiceN(1) }  // available at runtime
    .onSimulator { ... }
    .onDevice    { ... }
```

Cross-module wiring with `AutoRegistering`:

```swift
extension Container: @retroactive AutoRegistering {
    func autoRegister() {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hmlongco/Factory](https://github.com/hmlongco/Factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
