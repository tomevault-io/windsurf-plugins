---
trigger: always_on
description: This document provides context and conventions for AI coding agents (GitHub Copilot, Claude, Cursor, etc.) working with the Hex1b codebase.
---

# AI Coding Agent Guidelines for Hex1b

This document provides context and conventions for AI coding agents (GitHub Copilot, Claude, Cursor, etc.) working with the Hex1b codebase.

## 🛠️ Available Skills

This repository includes specialized skills in `.github/skills/` that provide detailed guidance for specific tasks. **Invoke these skills when working on related tasks** - they contain step-by-step procedures, templates, and best practices.

| Skill | When to Use |
|-------|-------------|
| **widget-creator** | Creating new widgets (widget records, nodes, theming, tests) |
| **writing-unit-tests** | Writing unit tests for widgets, nodes, or terminal functionality |
| **test-fixer** | Diagnosing flaky tests, especially timing-related failures in CI |
| **api-reviewer** | Reviewing API design, accessibility modifiers, and naming conventions |
| **doc-writer** | Writing XML API documentation or end-user guides |
| **doc-tester** | Validating documentation accuracy against library behavior |
| **surface-benchmarker** | Running performance benchmarks after modifying `src/Hex1b/Surfaces/` |
| **aspire** | Working with .NET Aspire (running samples, debugging, MCP tools) |

Skills are invoked automatically by AI agents based on the task context. They contain comprehensive procedures that complement the high-level guidance in this file.

## 📋 Project Overview

**Hex1b** is a .NET library for building terminal user interfaces (TUI) with a React-inspired declarative API. The library ships to NuGet as `Hex1b`.

### Key Technologies
- **.NET 10.0** (preview) - Target framework
- **C# 12+** - Modern C# features enabled
- **xUnit** - Testing framework
- **.NET Aspire** - Used for running sample applications

### Repository Layout
```
src/Hex1b/           → Main library (THE shipped package)
tests/Hex1b.Tests/   → Unit tests
samples/             → Example applications
apphost.cs           → Aspire app host for samples
```

## 🏗️ Architecture Concepts

### Widget/Node Pattern

Hex1b separates **configuration (widgets)** from **rendering (nodes)**:

| Concept | Location | Mutability | Purpose |
|---------|----------|------------|---------|
| **Widget** | `src/Hex1b/Widgets/` | Immutable | Describes what to render |
| **Node** | `src/Hex1b/Nodes/` | Mutable | Manages state, handles input, renders |

**Important**: Widgets are `record` types; Nodes are `class` types with mutable properties.

### Reconciliation

The `Hex1bApp.Reconcile()` method diffs widgets against nodes:
- Same widget type → Update existing node's properties
- Different widget type → Create new node
- This preserves state (focus, cursor position) across re-renders

### Layout System

Located in `src/Hex1b/Layout/`:
- `Constraints` - Min/max width/height bounds
- `Size` - Measured dimensions
- `Rect` - Position and size for arrangement
- `SizeHint` - Fill, Content, or Fixed sizing

### Render Loop (in Hex1bApp.cs)
```
Build widgets → Reconcile → Measure → Arrange → Render → Wait for input → Repeat
```

## 📝 Code Conventions

### Naming
- Widgets: `*Widget` (e.g., `TextBlockWidget`, `ButtonWidget`)
- Nodes: `*Node` (e.g., `TextBlockNode`, `ButtonNode`)
- State objects: `*State` (e.g., `TextBoxState`) - for widgets requiring user-owned mutable state

### Widget Definition Pattern
```csharp
// Widgets are records with fluent methods for event handlers
public record ButtonWidget(string Label) : Hex1bWidget
{
    internal Func<ButtonClickedEventArgs, Task>? ClickHandler { get; init; }
    
    public ButtonWidget OnClick(Action<ButtonClickedEventArgs> handler)
        => this with { ClickHandler = args => { handler(args); return Task.CompletedTask; } };
    
    public ButtonWidget OnClick(Func<ButtonClickedEventArgs, Task> handler)
        => this with { ClickHandler = handler };
}
```

### Node Definition Pattern
```csharp
public class ButtonNode : Hex1bNode
{
    // Properties reconciled from widget
    public string Label { get; set; } = "";
    public Func<InputBindingActionContext, Task>? ClickAction { get; set; }
    
    // Focus state (mutable, preserved across reconciliation)
    public bool IsFocused { get; set; }
    
    // Required overrides
    public override void Measure(Constraints constraints) { /* ... */ }
    public override void Arrange(Rect rect) { /* ... */ }
    public override void Render(Hex1bRenderContext context) { /* ... */ }
}
```

### Keybinding Rebinding Pattern

When widgets define input bindings in `ConfigureDefaultBindings`, use `Triggers(ActionId, handler, desc)` instead of `Action(handler, desc)` to make bindings rebindable:

```csharp
// In Widget record:
public static readonly ActionId MyAction = new($"{nameof(MyWidget)}.{nameof(MyAction)}");

// In Node's ConfigureDefaultBindings:
bindings.Key(Hex1bKey.Enter).Triggers(MyWidget.MyAction, handler, "Description");
```

Users can then remap/alias/disable actions via `WithInputBindings`:
```csharp
widget.WithInputBindings(b =>
{
    b.Remove(MyWidget.MyAction);  // remove default key
    b.Key(Hex1bKey.X).Triggers(MyWidget.MyAction);  // rebind to X
});
```

### Adding New Widgets


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mitchdenny/hex1b](https://github.com/mitchdenny/hex1b) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
