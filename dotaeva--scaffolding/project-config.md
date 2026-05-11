---
trigger: always_on
description: A reference for LLM coding agents (and humans) writing SwiftUI code with the **Scaffolding** library. Read this before generating navigation code in any project that uses Scaffolding.
---

# Scaffolding — Agent Guide

A reference for LLM coding agents (and humans) writing SwiftUI code with the **Scaffolding** library. Read this before generating navigation code in any project that uses Scaffolding.

The single most important idea: **Scaffolding's value is modular navigation across coordinator boundaries. The win is separation of concerns — UI views never own navigation state.** If you produce code that mixes navigation state into views, you've lost the reason to use the library.

---

## Why Scaffolding exists

SwiftUI's `NavigationStack(path:)` works for a single, self-contained screen graph. It breaks down once an app has:

- multiple feature modules that need to push into each other,
- destination types defined in different modules,
- coordinator-driven flows (login, onboarding, settings sheets),
- programmatic navigation that has to compose across module boundaries.

`NavigationStack` keeps navigation **inside the view tree**. That's the design constraint Scaffolding is built to escape. A `FlowCoordinatable` *is* a `NavigationStack` — but its destinations live on the coordinator (a plain Swift class), the macro generates the destination enum, and child coordinators slot in as routes without the view tree knowing.

If you find yourself reaching for `NavigationStack(path:)` inside a Scaffolding project, **stop**. There is almost certainly a coordinator-side answer.

---

## The hard rule: do not nest `NavigationStack`

`FlowCoordinatable` already wraps a `NavigationStack` internally. SwiftUI does **not** compose `NavigationStack`s with each other — the inner stack swallows pushes that should belong to the outer one, and `route(to:)` stops doing what you expect.

**Never put a `NavigationStack` inside any view returned by a `FlowCoordinatable` route function.** Not in the root view, not in a pushed detail view, not in a customise wrapper.

If a screen needs its own navigation hierarchy, give it a child coordinator:

```swift
// ❌ Wrong — nested NavigationStack breaks routing.
func detail(item: Item) -> some View {
    NavigationStack {       // ← don't.
        DetailRoot(item: item)
    }
}

// ✅ Right — child FlowCoordinator gets its own NavigationStack at the
//    coordinator boundary, where SwiftUI handles it correctly.
func detail(item: Item) -> any Coordinatable {
    DetailCoordinator(item: item)
}
```

The same applies to anything that wraps SwiftUI's stack: `NavigationView`, `NavigationSplitView`, custom containers that hold a `NavigationPath`. They all conflict.

---

## Picking a navigation primitive

When a user-facing transition needs to happen, use this decision tree:

```
Is it a push/pop on the current stack?
├─ Yes → coordinator.route(to: .someDestination)
│
└─ No, it's a modal.
   │
   Is the modal a single screen — confirmation, info dialog,
   simple form, picker?
   │
   ├─ Yes → SwiftUI native: .sheet(item:) / .fullScreenCover(item:)
   │
   └─ No, the modal contains its own navigation flow
      (multiple steps, push, dismiss-with-result, etc.).
      │
      → coordinator.present(.flow, as: .sheet)
        (returns a child coordinator from the route function)
```

### Concretely

| You want… | Use |
|---|---|
| Push a screen onto the current flow | `coordinator.route(to: .screen(args:))` |
| Pop the current screen | `coordinator.pop()` |
| Pop everything above the root | `coordinator.popToRoot()` |
| Show a confirmation dialog | SwiftUI's `.alert` / `.confirmationDialog` |
| Show a one-screen sheet (simple form, info) | SwiftUI's `.sheet(item:)` |
| Show a multi-step sub-flow | `coordinator.present(.subflow, as: .sheet)` |
| Show a full-screen sub-flow | `coordinator.present(.subflow, as: .fullScreenCover)` |
| Atomically replace the entire view hierarchy (auth, onboarding) | `appCoordinator.setRoot(.authenticated)` (on a `RootCoordinatable`) |
| Switch tabs programmatically | `tabCoordinator.selectFirstTab(.home)` |

Stay native for view-only modals. The native modifier is lighter, requires no coordinator boundary, and avoids the overhead of an extra `Destinations` case.

---

## Coordinator anatomy

```swift
@MainActor @Observable @Scaffoldable
final class HomeCoordinator: @MainActor FlowCoordinatable {
    // Required: the observable container that owns the stack.
    var stack = FlowStack<HomeCoordinator>(root: .home)

    // Routes — each becomes a `Destinations` enum case.
    func home()             -> some View         { HomeView() }
    func detail(item: Item) -> some View         { DetailView(item: item) }
    func settings()         -> any Coordinatable { SettingsCoordinator() }

    // Optional helpers (regular methods, not auto-generated).
    func openDetail(_ item: Item) {
        route(to: .detail(item: item))
    }
}
```

### Auto-tracked return types

The `@Scaffoldable` macro generates a `Destinations` enum with one case per function whose return type is one of:

| Return type | What it generates |
|---|---|
| `some View` | A view destination |
| `any Coordinatable` | A child-coordinator destination |
| `(any Coordinatable, some View)` | Tab tuple (coordinator + label) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dotaeva/scaffolding](https://github.com/dotaeva/scaffolding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
