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

## 2.1) Terminal reference implementations (required)

For any terminal-related work, first check the closest relevant behavior in:
- Windows Terminal / console host: https://github.com/microsoft/terminal
- Ghostty: https://github.com/ghostty-org/ghostty
- xterm.js: https://github.com/xtermjs/xterm.js
- PowerShell, when the issue involves PowerShell or Windows shell behavior:
  https://github.com/PowerShell/PowerShell

Terminal-related work includes PTY/ConPTY handling, VT parsing, terminal modes,
keyboard and mouse protocols, resize/reflow, rendering, text selection/copy,
palette/style behavior, hyperlinks, and graphics protocols.

For issues involving PowerShell, Windows shell startup, command invocation,
formatting/output, prompt behavior, environment handling, or ConPTY interaction
with PowerShell, inspect the PowerShell codebase before implementing a fix.
Document how PowerShell produces or expects the relevant behavior and how
RoyalTerminal should interoperate with it.

Before implementing or fixing behavior, compare these reference implementations
where relevant, decide which behavior RoyalTerminal should follow, and document
the decision in the plan, issue, PR notes, or tests. If RoyalTerminal must
diverge from a reference implementation, document the reason and cover the
chosen behavior with focused tests.

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

### Navigation (ReactiveUI routing) - (if requested)
- Use `IScreen` with a single `RoutingState` as the navigation root.
- All navigable ViewModels implement `IRoutableViewModel`.
- Views host navigation via `RoutedViewHost`.
- Use route segments that are stable, explicit, and testable.

### Avalonia integration
- Use `ReactiveUI.Avalonia` (latest) and do not use `Avalonia.ReactiveUI` directly.
- If a third-party dependency requires `Avalonia.ReactiveUI` (e.g., Dock integration),
  isolate it to the docking layer and do not reference it from app UI code.
  https://github.com/reactiveui/ReactiveUI.Avalonia


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [royalapplications/RoyalTerminal](https://github.com/royalapplications/RoyalTerminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
