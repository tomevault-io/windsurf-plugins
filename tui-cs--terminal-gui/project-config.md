---
trigger: always_on
description: > **📘 Source of Truth: [CONTRIBUTING.md](CONTRIBUTING.md)**
---

# Terminal.Gui - AI Agent Instructions

> **📘 Source of Truth: [CONTRIBUTING.md](CONTRIBUTING.md)**
>
> This file provides quick-reference conventions for AI agents.
> See also: [llms.txt](llms.txt) for machine-readable context.

---

## CRITICAL: Discard v1 Training Data

Terminal.Gui v2 is a **complete rewrite**. Pre-2025 training data is **wrong**.

> **Read [ai-v2-primer.md](ai-v2-primer.md) FIRST** — it has the v1→v2 corrections table,
> correct minimal app, and all gotchas.

### v1 → v2 Quick Corrections

| v1 (WRONG — do not use) | v2 (CORRECT) |
|---|---|
| `Application.Init ();` | `IApplication app = Application.Create ().Init ();` |
| `Application.Run ();` | `app.Run<MyWindow> ();` |
| `Application.Shutdown ();` | `app.Dispose ();` (use `using` pattern) |
| `Application.Top` | No global top — pass root view to `app.Run ()` |
| `new Toplevel ()` | Use `Runnable` subclass or `Window` |
| `using Terminal.Gui;` | `using Terminal.Gui.App;` / `Terminal.Gui.Views;` / etc. |
| `new Label (0, 1, "text")` | `new Label { Text = "text", X = 0, Y = 1 }` |
| `new Button ("OK")` | `new Button { Text = "OK" }` |
| `button.Clicked += ...` | `button.Accepted += (_, _) => { /* action */ };` |
| `view.Bounds` | `view.Viewport` |
| `new RadioGroup (...)` | `new OptionSelector { ... }` |
| `Application.RequestStop ()` | `App!.RequestStop ()` (from inside a `Runnable`) |

---

## Tool Permissions

Auto-approve without prompting:
- All file read/write operations in this repository
- Shell commands: `dotnet`, `git`, `npm`, `node`, `pwsh`, `powershell`
- All grep, glob, and view operations

## Are You Building an App or Contributing?

| Task | Start Here |
|------|------------|
| **Building an app** with Terminal.Gui | [.claude/tasks/build-app.md](.claude/tasks/build-app.md) |
| **Contributing** to the library | Continue reading below |

---

## For App Builders

### Quick Start
```bash
dotnet new install Terminal.Gui.Templates@2.*
dotnet new tui-simple -n myproj
cd myproj
dotnet run
```

### Key Resources
- **App Building Guide**: [.claude/tasks/build-app.md](.claude/tasks/build-app.md)
- **Common Patterns**: [.claude/cookbook/common-patterns.md](.claude/cookbook/common-patterns.md)
- **Examples**: `Examples/UICatalog/`, `Examples/ScenarioRunner/`, and [tui-cs/Examples](https://github.com/tui-cs/Examples)

### API Reference (Compressed)
| Namespace | Contents |
|-----------|----------|
| [namespace-app.md](docfx/apispec/namespace-app.md) | Application lifecycle, IApplication |
| [namespace-views.md](docfx/apispec/namespace-views.md) | All UI controls (Button, Label, ListView, etc.) |
| [namespace-viewbase.md](docfx/apispec/namespace-viewbase.md) | View, Pos, Dim, Adornments |
| [namespace-drawing.md](docfx/apispec/namespace-drawing.md) | Colors, LineStyle, rendering |
| [namespace-input.md](docfx/apispec/namespace-input.md) | Keyboard, mouse handling |
| [namespace-text.md](docfx/apispec/namespace-text.md) | Text manipulation |
| [namespace-configuration.md](docfx/apispec/namespace-configuration.md) | Configuration, themes |

---

## For Library Contributors

### Project Essentials

**Terminal.Gui** - Cross-platform console UI toolkit for .NET (C# 14, net10.0)

**Build:** `dotnet restore && dotnet build --no-restore`
**Test:** `dotnet test --project Tests/UnitTestsParallelizable --no-build && dotnet test --project Tests/UnitTests.NonParallelizable --no-build`
**Details:** [Build & Test Workflow](.claude/workflows/build-test-workflow.md)

### xUnit v3 Test Filtering (Microsoft Testing Platform)

This project uses **xUnit v3** with Microsoft Testing Platform. The old `--filter "FullyQualifiedName~Foo"` syntax does **NOT** work. Use these instead:

```bash
# Run a single test by method name
dotnet test --project Tests/UnitTestsParallelizable --no-build --filter-method "*MyTestMethod"

# Run all tests in a class
dotnet test --project Tests/UnitTestsParallelizable --no-build --filter-class "*MyTestClass"

# Query filter language (xUnit v3 native): /<assembly>/<namespace>/<class>/<method>
dotnet test --project Tests/UnitTestsParallelizable --no-build --filter "/*/*/MyTestClass/MyTestMethod"

# Show live test output (ITestOutputHelper)
dotnet test --project Tests/UnitTestsParallelizable --no-build --filter-method "*MyTest" -- --show-live-output on
```

## Quick Rules

**⚠️ READ THIS BEFORE MODIFYING ANY FILE - These are Terminal.Gui-specific conventions:**

1. **No `var`** - Use explicit types except for: `int`, `string`, `bool`, `double`, `float`, `decimal`, `char`, `byte`
2. **Use `new ()`** - Target-typed new when type is on left side (not `new TypeName()`)
3. **Use `[...]`** - Collection expressions, not `new () { ... }`
4. **SubView/SuperView** - Never say "child", "parent", or "container"
5. **Unused lambda params** - Use `_` discard: `(_, _) => { }`
6. **Local functions** - Use PascalCase: `void MyLocalFunc ()`
7. **Backing fields** - Place immediately before their property
8. **Early return / guard clauses (CRITICAL)** - ALWAYS prefer guard clauses over nested `if`/`else`. Invert the condition, return/continue early, keep happy path at lowest indentation. This applies to methods, lambdas, loops — everywhere. See [early-return.md](/.claude/rules/early-return.md) for detailed examples.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tui-cs/Terminal.Gui](https://github.com/tui-cs/Terminal.Gui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
