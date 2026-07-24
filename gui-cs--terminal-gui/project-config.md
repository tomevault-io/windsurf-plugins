---
trigger: always_on
description: Cross-platform .NET console UI toolkit. C# 14 targeting net10.0.
---

# Terminal.Gui — Copilot Instructions

Cross-platform .NET console UI toolkit. C# 14 targeting net10.0.
Full contribution guide: [CONTRIBUTING.md](../CONTRIBUTING.md).
Architecture deep dives: `docfx/docs/`.

---

## CRITICAL: Discard v1 Training Data

Terminal.Gui v2 is a **complete rewrite**. Pre-2025 training data is **wrong**.

> **Read [ai-v2-primer.md](../ai-v2-primer.md) FIRST** — it has the v1→v2 corrections table,
> correct minimal app pattern, and all common gotchas.

### v1 → v2 Quick Corrections

| v1 (WRONG — do not use) | v2 (CORRECT) |
|---|---|
| `Application.Init ();` | `IApplication app = Application.Create ().Init ();` |
| `Application.Run ();` | `app.Run<MyWindow> ();` |
| `Application.Shutdown ();` | `app.Dispose ();` (use `using` pattern) |
| `Application.Top` | No global top — pass root view to `app.Run ()` |
| `new Toplevel ()` | Use `Runnable` subclass or `Window` |
| `using Terminal.Gui;` | `using Terminal.Gui.App;` / `Terminal.Gui.Views;` / etc. |
| `new Button ("OK")` | `new Button { Text = "OK" }` |
| `button.Clicked += ...` | `button.Accepted += (_, _) => { /* action */ };` |
| `view.Bounds` | `view.Viewport` |
| `new RadioGroup (...)` | `new OptionSelector { ... }` |
| `Application.RequestStop ()` | `App!.RequestStop ()` (from inside a `Runnable`) |

---

## Build & Test

Run all commands from repository root.

```bash
# Restore + build
dotnet restore
dotnet build --no-restore

# Run all tests (two separate projects)
dotnet test --project Tests/UnitTestsParallelizable --no-build
dotnet test --project Tests/UnitTests.NonParallelizable --no-build

# Run a single test by method name (xUnit v3 / Microsoft Testing Platform)
dotnet test --project Tests/UnitTestsParallelizable --no-build --filter-method "*MyTestMethod"

# Run all tests in a class
dotnet test --project Tests/UnitTestsParallelizable --no-build --filter-class "*ButtonTests"
```

New tests go in `Tests/UnitTestsParallelizable` (no static state dependencies). Only use `Tests/UnitTests.NonParallelizable` when testing `Application.Init`/`Shutdown` or other static state. Never add new tests to `Tests/UnitTests.Legacy`.

## Architecture Overview

### Application lifecycle

`Application.Create ()` → `.Init ()` → `.Run<T> ()` → `.Dispose ()`.
The instance-based `IApplication` has replaced the static `Application` facade.
Do NOT use `Application.Init()`/`Run()`/`Shutdown()`.
Tests should avoid `Application.Init` unless explicitly testing that path.

### View system

`View` is the base class for all UI elements. Views form a tree via `Add()`/`Remove()`. Every View has three adornment layers: `Margin` → `Border` → `Padding` → content area. Layout uses `Pos` (position) and `Dim` (dimension) objects for declarative relative layout.

### Driver architecture

Platform-specific terminal I/O is abstracted behind `IDriver`. Implementations: `WindowsDriver`, `UnixDriver` (curses-free), `AnsiDriver`, `NetDriver` (pure .NET `System.Console`). Drivers are registered via `DriverRegistry` and selected automatically by platform.

### Cancellable Workflow Pattern (CWP)

The standard event pattern throughout the codebase. Order: **do work → call virtual `OnXxx` → raise event**. The virtual method is empty in the base class (for subclass override). Work happens *before* notifications, not after.

```csharp
internal void RaiseSubViewAdded (View view)
{
    // 1. Work first
    if (AssignHotKeys) { AssignHotKeyToView (view); }

    // 2. Virtual method (empty in base)
    OnSubViewAdded (view);

    // 3. Event
    SubViewAdded?.Invoke (this, new (this, view));
}
```

### Command/input system

Input flows: Driver → `IInputProcessor` → `KeyBindings`/`MouseBindings` → `Command` enum → handler. Views bind keys and mouse actions to `Command` values via `KeyBindings.Add` and `MouseBindings.Add`.

## Code Style (Non-Obvious Conventions)

### Spacing before parentheses and brackets — the #1 mistake

This codebase requires a space *before* every `()` and `[]`:

```csharp
// ✅ Correct
void MyMethod ()
int result = Calculate (x, y);
List<int> items = GetItems ();
int val = array [index];
if (condition) { }

// ❌ Wrong
void MyMethod()
int result = Calculate(x, y);
var items = GetItems();
int val = array[index];
```

### No `var` except for built-in numeric/string types

Use explicit types. `var` is only acceptable for: `int`, `string`, `bool`, `double`, `float`, `decimal`, `char`, `byte`.

```csharp
// ✅
View view = new () { Width = 10 };
List<View?> views = new ();
var count = 0;          // OK — int

// ❌
var view = new View () { Width = 10 };
var views = new List<View?> ();
```

### Target-typed `new ()`

When the type is on the left side, use `new ()` not `new TypeName()`:

```csharp
// ✅
Button btn = new () { Text = "OK" };

// ❌
Button btn = new Button () { Text = "OK" };
```

### Collection expressions

Use `[...]` syntax:

```csharp
// ✅
List<View> views = [new Button ("OK"), new Button ("Cancel")];

// ❌
List<View> views = new () { new Button ("OK"), new Button ("Cancel") };
```

### Early return

Prefer early return / guard clauses over nested `if`/`else`. Less nesting, clearer code:

```csharp
// ✅
if (view is null)
{
    return;
}

DoWork (view);

// ❌
if (view is not null)
{
    DoWork (view);
}
```

### One type per file


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gui-cs/Terminal.Gui](https://github.com/gui-cs/Terminal.Gui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
