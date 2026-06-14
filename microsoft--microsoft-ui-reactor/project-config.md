---
trigger: always_on
description: Reactor is a declarative, component-based C# framework for building WinUI 3 desktop apps. It renders real WinUI controls via a virtual element tree and reconciler — similar to React's programming model but targeting native Windows UI.
---

# Copilot Instructions — Microsoft.UI.Reactor

Reactor is a declarative, component-based C# framework for building WinUI 3 desktop apps. It renders real WinUI controls via a virtual element tree and reconciler — similar to React's programming model but targeting native Windows UI.

## Build, Test, Lint

```bash
# Build (platform defaults to machine arch for apps; libraries are AnyCPU)
dotnet build Reactor.slnx

# Unit tests — xUnit, headless, fast (~2200 tests incl. 590 Yoga fixtures)
dotnet test tests/Reactor.Tests

# Single test class
dotnet test tests/Reactor.Tests --filter "FullyQualifiedName~ReconcilerMountUpdateTests"

# Selftests — real WinUI window, in-process (~10s)
dotnet test tests/Reactor.SelfTests

# Raw TAP output (faster iteration, supports --filter prefix)
dotnet run --project tests/Reactor.AppTests.Host -- --self-test --filter "Flex"

# E2E — Appium/WinAppDriver (requires WinAppDriver installed)
dotnet test tests/Reactor.AppTests

# Single E2E class
dotnet test tests/Reactor.AppTests --filter "ClassName=Reactor.AppTests.Tests.AccessibilityTests"
```

CI runs unit tests + selftests + full solution build on every PR. .NET 10 SDK, `windows-latest` runner.

Full testing guide — tier selection, NativeAOT runs, code coverage — in [`TESTING.md`](TESTING.md).

## Architecture

### Virtual DOM model

UI is described as **immutable C# records** (`Element` subclasses), not WinUI controls. The reconciler diffs old vs. new element trees and patches only what changed on real controls.

```
Component.Render() → Element tree (records)
                        ↓
                   Reconciler
                   ├── Mount  → creates WinUI controls
                   └── Update → diffs & patches controls
```

### Reconciler is split across partial classes

- `Reconciler.cs` — orchestration, child reconciliation, unmount, helpers
- `Reconciler.Mount.cs` — `MountXxx()` handler per control type
- `Reconciler.Update.cs` — `UpdateXxx()` handler per control type

### Hooks follow React rules

Hooks (`UseState`, `UseEffect`, `UseReducer`, `UseMemo`, etc.) are tracked by call order in `RenderContext`. They must be called unconditionally, in the same order every render — no conditional hooks. Pass `threadSafe: true` for cross-thread state updates.

### Echo suppression for value controls

Echo handling is a documented hybrid (spec-047 §8.3). Synchronous, exact-comparable, single-controlled-value round-trips (ComboBox, FlipView, GridView, ListBox, Pivot, PipsPager, RadioButtons, SelectorBar, TabView, TemplatedFlipView, ToggleSwitch, TextBox) use a value-diff arm (`ReactorState.PendingEchoMatch` + `ArmExpectedEcho`/`ShouldSuppressEcho`, opt-in `valueDiffEcho`). `ChangeEchoSuppressor` is **retained** as the suppress-counter fallback for the rest: doubles (Slider/NumberBox value), NumberBox coercion, CalendarView collection diff, deferred/coercion strings (AutoSuggest/Password/RichEdit), Expander, CheckBox path-B, the `ApplySetters` suppression scope, and the public `WriteSuppressed` primitive. Authors keep using the stable `WriteSuppressed` primitive (or declare `.Controlled` / `valueDiffEcho` on a descriptor) — never the suppressor directly.

### Element pooling

`ElementPool` recycles WinUI controls. Poolable types track one-time event wiring via `ConditionalWeakTable<FrameworkElement, PoolableWireFlags>` to avoid double-subscribing across rent/return cycles.

### Per-element state via attached DP

`ReactorAttached.StateProperty` stores `ReactorState` (Element pointer + `ModifierEventHandlerState` — the routed-input family, lazily allocated — + per-control `ControlEventStateBox` for control-intrinsic events) on native elements — not `FrameworkElement.Tag` or a CWT.

## Key Conventions

### Elements are immutable records

```csharp
public record MyControlElement(string Label, Action? OnClick = null) : Element;
```

Use `with` expressions for variations. Never mutate.

### Factory methods over constructors

The DSL entry point is `using static Microsoft.UI.Reactor.Factories;`. Factory methods return Element records, never WinUI controls:

```csharp
TextBlock("hello")       // not new TextBlockElement("hello")
Button("+", () => ...)   // not new ButtonElement(...)
VStack(child1, child2)   // layout containers
```

`Factories` is `public static partial class` — factory methods can be added from multiple files.

### Fluent modifiers preserve concrete types

Extension methods use `<T> where T : Element` to maintain the concrete type through chains:

```csharp
Text("Hello").Bold().Margin(16).Set(tb => tb.TextWrapping = TextWrapping.Wrap)
// Still TextBlockElement throughout the chain
```

### Adding a new WinUI control

The legacy Element-record + `MountXxx`/`UpdateXxx` dispatch-switch path is gone. The current path:

1. **Element record** in `src/Reactor/Core/Element.cs`
2. **Authoring shape** — a `ControlDescriptor<TElement, TControl>` (the primary path) or a hand-coded `IElementHandler<TElement, TControl>` for irregular controls.
3. **Register** it in `RegisterV1BuiltInHandlers`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/microsoft-ui-reactor](https://github.com/microsoft/microsoft-ui-reactor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
