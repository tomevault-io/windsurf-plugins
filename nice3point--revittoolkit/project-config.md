---
trigger: always_on
description: * **`/source`**: Core library project.
---

# RevitToolkit Guidelines

## 1. Project Structure

### 1.1. Solution Organization

* **`/source`**: Core library project.
    * `Nice3point.Revit.Toolkit`: Toolkit classes for Revit API (contexts, external commands, handlers, options, helpers).
* **`/tests`**: Testing projects.
    * `Nice3point.Revit.Toolkit.Tests`: Unit tests executed in Revit context using Nice3point.TUnit.Revit.
* **Root Level**:
    * Configuration files: `Directory.Build.props`, `Directory.Packages.props`
    * Documentation: `Readme.md`, `Changelog.md`
    * CI/CD: `.github/workflows`

## 2. Architecture Principles

### 2.1. Core Design Goals

* **Static Context Access:** Provide global access to Revit Application and UI contexts.
* **Simplification:** Abstract boilerplate code for Revit interfaces (`IExternalCommand`, `IExternalApplication`, etc.).
* **Async Support:** Enable async/await patterns within Revit external commands and event handlers.
* **Thread Safety:** Ensure thread-safe operations for shared state.
* **Disposable Scopes:** Use `IDisposable` pattern for resource management.
* **Analyzers:** Use JetBrains.Annotations for static code analysis.
* **Backward Compatibility:** Never break existing public APIs.

## 3. Strict C# Production Style

### 3.1. General Principles

* **Instance Classes:** Use instance-based classes with inheritance for external commands/applications.
* **Static Context Classes:** Use static classes for global context access.
* **Disposable Scope Pattern:** Return `IDisposable` for scoped operations.
* **Pure Functions:** Mark read-only operations with `[Pure]` attribute where applicable.
* **Explicit over Implicit:** Code should be self-explanatory.

### 3.2. Naming Conventions

* **Clarity is King:** Names must be descriptive.
* **Revit API Patterns:** Follow Revit API naming conventions.
* **Scope Methods:** Use `Begin...Scope` pattern for disposable scopes:
    * ✅ `BeginFailureSuppressionScope()`, `BeginDialogSuppressionScope()`
    * ❌ `SuppressFailures()`, `SuppressDialogs()`
* **No Abbreviations:**
    * ❌ `elem`, `doc`, `param`, `app`
    * ✅ `element`, `document`, `parameter`, `application`

### 3.3. Class Structure

* **File-Scoped Namespaces:** Always use `namespace Nice3point.Revit.Toolkit;` or sub-namespaces.
* **PublicAPI Attribute:** Mark all public classes with `[PublicAPI]`.
* **EditorBrowsable:** Mark callback methods with `[EditorBrowsable(EditorBrowsableState.Never)]`.
* **XML Documentation:** Document all public members with `<summary>`, `<param>`, `<returns>`, `<remarks>`, `<example>` blocks.

### 3.4. Disposable Scope Pattern

All scoped operations should return `IDisposable` for automatic resource management:

```csharp
public static IDisposable BeginSomeScope()
{
    lock (SomeLock)
    {
        if (_scopeCount++ == 0)
        {
            // Subscribe to events or acquire resources
        }
    }

    return new SomeScope();
}

private sealed class SomeScope : IDisposable
{
    private int _disposed;

    public void Dispose()
    {
        if (Interlocked.Exchange(ref _disposed, 1) != 0) return;

        lock (SomeLock)
        {
            if (--_scopeCount == 0)
            {
                // Unsubscribe from events or release resources
            }
        }
    }
}
```

### 3.5. External Command/Application Pattern

Base classes should handle boilerplate and provide simplified override methods:

```csharp
public abstract class SomeExternalCommand : IExternalCommand
{
    public Result Result { get; set; } = Result.Succeeded;
    
    [EditorBrowsable(EditorBrowsableState.Never)]
    public Result Execute(ExternalCommandData commandData, ref string message, ElementSet elements)
    {
        // Setup and assembly resolution
        using (ResolveHelper.BeginAssemblyResolveScope(GetType()))
        {
            Execute();
        }
        return Result;
    }

    public abstract void Execute();
}
```

### 3.6. Async Command Pattern

Use `DispatcherFrame` for message pumping in async commands:

```csharp
public sealed override void Execute()
{
    var task = ExecuteAsync();
    if (task.IsCompleted)
    {
        task.GetAwaiter().GetResult();
        return;
    }

    var frame = new DispatcherFrame();
    task.ContinueWith(_ => frame.Continue = false, TaskScheduler.Default);
    Dispatcher.PushFrame(frame);
    task.GetAwaiter().GetResult();
}
```

### 3.7. Event Handler Pattern

Use `ConcurrentQueue` for thread-safe action queuing:

```csharp
private readonly ConcurrentQueue<Action<UIApplication>> _queue = new();

public void Raise(Action<UIApplication> action)
{
    if (RevitContext.IsRevitInApiMode)
    {
        action(RevitContext.UiApplication);
        return;
    }

    _queue.Enqueue(action);
    Raise();
}
```

### 3.8. Error Handling

* **Revit Exceptions:** Document Revit API exceptions in XML comments.
* **No Swallowing:** Let Revit exceptions propagate to caller by default.
* **Optional Exception Handler:** Provide `SetExceptionHandler()` for handlers that process multiple actions.
* **Validation:** Use `ThrowWhen()` pattern for internal validation.

### 3.9. Thread Safety

* **Lock Objects:** Use `Lock` class for synchronization.
* **Interlocked Operations:** Use `Interlocked.Exchange` for dispose flags.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nice3point/RevitToolkit](https://github.com/Nice3point/RevitToolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
