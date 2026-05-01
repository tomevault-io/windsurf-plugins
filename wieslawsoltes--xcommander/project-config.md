---
trigger: always_on
description: This document defines the architectural and coding rules for the project. It is
---

# Engineering Guide (AGENTS)

This document defines the architectural and coding rules for the project. It is
authoritative for all new code and refactors.

## 1) Core principles (non-negotiable)

### SOLID (strict)
- Single Responsibility: every class has exactly one reason to change.
- Open/Closed: extend behavior via composition and interfaces; avoid modifying
  stable code paths when adding features.
- Liskov Substitution: derived types must be safely substitutable without
  altering expected behavior or contract.
- Interface Segregation: prefer small, focused interfaces; avoid "god" interfaces.
- Dependency Inversion: depend on abstractions; wire concrete types in the
  composition root only.

### MVVM (strict)
- Views are passive. No UI logic in code-behind beyond `InitializeComponent()`.
- All inputs are routed to ViewModels via bindings, commands, and behaviors.
- ViewModels are UI-framework agnostic and unit-testable.
- Models and services contain business logic and data access; ViewModels orchestrate
  them via DI.
- Prefer composition in ViewModels/services/code over inheritance wherever possible,
  except where framework base types are required (e.g., `ReactiveObject` for ViewModels).

## 2) Architecture

### Layering
- UI (Avalonia Views + XAML): visual composition only.
- Presentation (ViewModels): state, commands, reactive composition.
- Domain/Services: business logic, parsing, validation, domain rules.
- Infrastructure: file system, persistence, external integrations.

### Boundaries
- UI depends on Presentation; Presentation depends on Domain; Infrastructure is
  depended on by Domain or Presentation via interfaces.
- No reference from Domain to UI or Avalonia types.

## 3) Avalonia UI best practices (aligned with Avalonia codebase)

Reference: https://github.com/AvaloniaUI/Avalonia

### Views and styling
- Use XAML for layout and visuals; avoid creating controls in code.
- Define styles and resources in dedicated resource dictionaries and merge them
  in `App.axaml` to keep styling consistent and maintainable.
- Prefer `StaticResource` for immutable resources and `DynamicResource` when
  runtime updates are required.

### Data binding
- Use compiled bindings only (no reflection bindings) with explicit `x:DataType` on
  all binding scopes (views, DataTemplates, control themes, and resources).
- Keep bindings one-way unless user input must update the ViewModel.
- Use `DataTemplates` or a `ViewLocator` (custom, non-reflection) for view lookup.

### Custom controls
- Use `StyledProperty` only for values that must participate in styling.
- Prefer `DirectProperty` for non-styled properties to avoid extra overhead.
- For best UI/UX, prefer custom control creation or re-templating using control themes
  instead of CRUD-style UI.

## 4) ReactiveUI (required)

Reference: https://github.com/reactiveui/ReactiveUI

### ViewModel base
- All ViewModels inherit from `ReactiveObject`.
- Use `ReactiveCommand` for commands; never use event handlers in code-behind.
- Use `WhenAnyValue`, `ObservableAsPropertyHelper`, and `Interaction<TIn,TOut>`
  to model state, derived values, and dialogs.
- Use `ReactiveUI.SourceGenerators` for INPC/ReactiveObject boilerplate where applicable.
  https://github.com/reactiveui/ReactiveUI.SourceGenerators

### Navigation (ReactiveUI routing)
- Use `IScreen` with a single `RoutingState` as the navigation root.
- All navigable ViewModels implement `IRoutableViewModel`.
- Views host navigation via `RoutedViewHost`.
- Use route segments that are stable, explicit, and testable.

### Avalonia integration
- Use `ReactiveUI.Avalonia` (latest) and do not use `Avalonia.ReactiveUI` directly.
- If a third-party dependency requires `Avalonia.ReactiveUI` (e.g., Dock integration),
  isolate it to the docking layer and do not reference it from app UI code.
  https://github.com/reactiveui/ReactiveUI.Avalonia

## 5) Input and interaction via Xaml.Behaviors (required)

Reference: https://github.com/wieslawsoltes/Xaml.Behaviors

- All UI input and events are handled via behaviors/triggers.
- Prefer source-generator-based behaviors/actions (no reflection) wherever available.
- Use trigger behaviors (property, data, loaded/unloaded, routed event) for
  lifecycles and state transitions.
- Code-behind must not contain event handlers or direct ViewModel calls.

## 6) Docking layout with Dock for Avalonia (required)

Reference: https://github.com/wieslawsoltes/Dock

- Use Dock.Model.* to represent the docking layout state.
- Use Dock.Avalonia for the view layer and Dock.Model.ReactiveUI for MVVM
  integration.
- Persist layout state to user settings and restore on startup.
- Keep layout logic in ViewModels; Views only render the Dock model.

## 7) Text editing with AvaloniaEdit (required)

Reference: https://github.com/AvaloniaUI/AvaloniaEdit

- Use AvaloniaEdit `TextEditor` for all code/text editing surfaces.
- Enable syntax highlighting using TextMate grammars/themes.
- Keep editor configuration in ViewModels (options, text, selection) and bind to
  the view.

## 8) Data presentation with ProDataGrid (required)

Reference: https://github.com/wieslawsoltes/ProDataGrid

- Use ProDataGrid `DataGrid` for all tabular data, tree views, and list displays.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wieslawsoltes/XCommander](https://github.com/wieslawsoltes/XCommander) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
