---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Coding Style

### Naming

| Element | Convention | Example |
|---|---|---|
| Public properties | PascalCase | `AllLogCount`, `IsActivated` |
| Private fields (instance and static) | camelCase; instance fields always qualified with `this.` | `this.logs`, `latestPanelSizeKey`, `defaultTimestampCultureInfo` |
| Internal / protected / public fields | PascalCase | `TimestampAsc` |
| Static `ObservableProperty` fields | PascalCase + `Prop` suffix | `AllLogCountProp` |
| Private/helper methods | PascalCase | `ChangeState()`, `OnDataSourcePropertyChanged()` |
| Public methods | PascalCase | `ClearLogs()`, `Pause()` |
| Async methods | Must end with `Async` | `OnPrepareShuttingDownAsync()` |
| Constants | PascalCase | `DefaultSidePanelSize` |
| Parameters & local variables | camelCase | `cancellationToken`, `logReader` |
| Interfaces | `I` prefix + PascalCase | `ILogDataSource`, `IDisplayableLogProcessor` |
| Event handlers | `On` prefix | `OnDataSourcePropertyChanged` |

**All methods use PascalCase regardless of accessibility**, per the standard .NET naming convention — a private or internal helper method is named exactly like a public one (`ChangeState()`, not `changeState()`). camelCase is reserved for private fields, parameters, and local variables; there is no case in which a method is camelCase. Some existing types still use camelCase for their private methods — that is legacy, not a convention to match, and new methods in those types are PascalCase too.

**Field casing follows accessibility, not lifetime** — a field is camelCase when it is `private` (instance or static alike) and PascalCase when it is `internal`, `protected`, or `public`. When adding a field to an existing type, follow the convention already used in that type.

### Formatting & Structure

- **File-scoped namespaces** — use `namespace Foo.Bar;` (not block-scoped).
- **`using` directives outside** the namespace declaration. Always import the correct namespace when using a new type; after any code modification, audit every edited file and remove `using` directives that are no longer referenced.
- **Allman-style braces** — opening brace on its own line for types and methods; single-statement bodies may omit braces, but only when that single statement fits on one line. If the inner statement spans multiple lines (e.g. the outer of a stacked `using` whose inner `using` has a multi-line block), the outer statement must use braces.
- **`try`/`catch`/`finally` blocks** always use full braces even when the body is a single statement or empty.
- **`this.` prefix** on all instance member accesses (fields and properties). It does **not** apply to primary-constructor parameters, which are accessed directly by name.
- **Static members are accessed through the type that declares them**, never through a derived type that merely inherits them. `CurrentOrNull` is declared on `CarinaStudio.Application`, which `App` inherits it from, so write `Application.CurrentOrNull` — **not** `App.CurrentOrNull`. Both spellings bind to the identical member, so this is purely about showing the reader where the member actually lives, and about not implying that the derived type adds something it does not. `using CarinaStudio;` resolves the bare `Application` name; qualify it in a file that also imports `Avalonia`, where `Avalonia.Application` would collide.
- **Primary constructors** preferred over explicit constructors when the body would only assign fields.
- **Expression-bodied members** for concise single-expression properties and methods.
- **Assignments are dedicated statements** — never combine an assignment with a value read in the same expression. Do not consume the result of an assignment (`=`, `??=`, `++`, `--`, etc.) as a sub-expression (method argument, condition, initializer, return value, expression-bodied member, etc.). Assign on its own line first, then read the variable/field on the following line. This also rules out returning an assignment: a lazily-initialized property must use a block getter that assigns on one line and returns on the next (`get { field ??= …; return field; }`), **not** an expression body that consumes the assignment (`=> field ??= …`).
- **Enum members** are listed consecutively with no blank line between them, even when each carries an XML doc comment.
- **Blank lines between members** — two blank lines between members of a top-level type; one blank line between members of an inner (nested) type.

### Fields & Properties

- Access instance fields with `this.` consistently — never omit it.
- Mark thread-shared fields `volatile`; use `Interlocked.*` for atomic updates.
- Use `[ThreadSafe]` / `[UsedOnBackgroundThread]` / `[CalledOnBackgroundThread]` attributes to document thread semantics.
- Register reactive properties with `ObservableProperty.Register<TOwner, T>(nameof(...))` rather than implementing `INotifyPropertyChanged` manually. Coercion logic goes in the registration call.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carina-studio/ULogViewer](https://github.com/carina-studio/ULogViewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
