---
trigger: always_on
description: **You are an expert AI assistant working on the Foundatio.Mediator source generator. This document defines the exact rules and patterns you must follow when modifying code generation logic.**
---

# AI Instructions: Foundatio.Mediator Source Generator

**You are an expert AI assistant working on the Foundatio.Mediator source generator. This document defines the exact rules and patterns you must follow when modifying code generation logic.**

## Critical Principles

1. **Never break existing semantics** - Generated code must maintain backward compatibility
2. **Follow the patterns exactly** - Handler/middleware instantiation follows strict rules based on lifetime
3. **Performance is critical** - Avoid allocations, use aggressive inlining, cache where safe
4. **Test thoroughly** - All changes require running `dotnet build` then `dotnet test`
5. **Static caching rules**:
   - Only cache when handler/middleware has NO constructor dependencies AND lifetime is None/Default
   - NEVER cache when handler/middleware has constructor dependencies (use `ActivatorUtilities.CreateInstance` each invocation - deps may be scoped)
   - NEVER cache when lifetime is Scoped/Transient (always resolve from DI)
   - NEVER cache when lifetime is Singleton (always resolve from DI - let DI container manage singleton caching)

## Handler Instantiation Rules (CRITICAL)

The `GenerateGetOrCreateHandler` and `EmitHandlerInvocation` methods MUST follow these exact patterns:

### 1. Static Handlers

**No caching needed** - call directly:

```csharp
// In EmitHandlerInvocation:
string accessor = handler.FullName;
source.AppendLine($"{asyncModifier}{accessor}.{handler.MethodName}({parameters});");
```

### 2. Scoped/Transient Lifetime

**Always resolve from DI - NO caching, NO generated method**:

```csharp
// Check in EmitHandlerInvocation:
if (handler.RequiresDIResolutionPerInvocation)
{
    source.AppendLine($"var handlerInstance = serviceProvider.GetRequiredService<{handler.FullName}>();");
    accessor = "handlerInstance";
}
```

**Important**: `RequiresDIResolutionPerInvocation` returns true when:

- `handler.Lifetime` is `"Scoped"`, OR
- `handler.Lifetime` is `"Transient"`, OR
- Assembly-level `MediatorConfiguration.HandlerLifetime` is `Scoped` or `Transient` AND `handler.Lifetime` is `None`

### 3. Explicit Singleton Lifetime

**Always resolve from DI - NO static caching**:

```csharp
// Check in EmitHandlerInvocation:
if (string.Equals(handler.Lifetime, "Singleton", StringComparison.OrdinalIgnoreCase))
{
    source.AppendLine($"var handlerInstance = serviceProvider.GetRequiredService<{handler.FullName}>();");
    accessor = "handlerInstance";
}
```

**Why no static caching for Singleton?**

- User explicitly wants DI to manage the instance
- Let the DI container handle singleton caching (it's optimized for this)
- Avoids potential issues with multiple ServiceProvider instances (e.g., in tests)

### 4. No Constructor Dependencies, Lifetime is None/Default

**Static cached instance with lazy initialization**:

```csharp
// In GenerateGetOrCreateHandler:
private static {handler.FullName}? _cachedHandler;

[DebuggerStepThrough]
[MethodImpl(MethodImplOptions.AggressiveInlining)]
private static {handler.FullName} GetOrCreateHandler(IServiceProvider serviceProvider)
{
    return _cachedHandler ??= new {handler.FullName}();
}

// In EmitHandlerInvocation:
source.AppendLine("var handlerInstance = GetOrCreateHandler(serviceProvider);");
accessor = "handlerInstance";
```

### 5. Has Constructor Dependencies, Lifetime is None/Default

**Use `ActivatorUtilities.CreateInstance` - NO caching**:

```csharp
// In EmitHandlerInvocation:
source.AppendLine($"var handlerInstance = ActivatorUtilities.CreateInstance<{handler.FullName}>(serviceProvider);");
accessor = "handlerInstance";
```

**Important**: When a handler has constructor dependencies AND lifetime is None/Default, a fresh instance is created via `ActivatorUtilities.CreateInstance` on every invocation. We do NOT cache these handlers because their constructor dependencies may be scoped (e.g., `DbContext`, `IMediator`) and would become stale or invalid after the scope ends.

## Middleware Instantiation Rules (CRITICAL)

The `GenerateMiddlewareInstantiation` and `EmitMiddlewareInstances` methods MUST follow these exact patterns:

### 1. Static Middleware

**No instantiation needed** - call directly:

```csharp
// In middleware invocation code:
string accessor = middleware.FullName;
```

### 2. Scoped/Transient Lifetime

**Always resolve from DI - NO caching, NO generated method**:

```csharp
// In EmitMiddlewareInstances:
if (m.RequiresDIResolutionPerInvocation)
{
    source.AppendLine($"var {varName} = serviceProvider.GetRequiredService<{m.FullName}>();");
}
```

### 3. Explicit Singleton Lifetime

**Always resolve from DI - NO static caching**:

```csharp
// In EmitMiddlewareInstances:
if (string.Equals(m.Lifetime, "Singleton", StringComparison.OrdinalIgnoreCase))
{
    source.AppendLine($"var {varName} = serviceProvider.GetRequiredService<{m.FullName}>();");
}
```

### 4. No Constructor Dependencies, Lifetime is None/Default

**Static cached instance with lazy initialization**:

```csharp
// In GenerateMiddlewareInstantiation:
private static {m.FullName}? _cached{m.Identifier};

[DebuggerStepThrough]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FoundatioFx/Foundatio.Mediator](https://github.com/FoundatioFx/Foundatio.Mediator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
