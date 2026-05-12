---
trigger: always_on
description: Provides: [Expressive], [ExpressiveFor], [ExpressiveForConstructor], [PolyfillTarget],
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

ExpressiveSharp is a Roslyn source generator that enables modern C# syntax (null-conditional `?.`, switch expressions, pattern matching) in LINQ expression trees, which normally only support a restricted subset of C#. It works by generating `Expression<TDelegate>` factory code at compile time from `[Expressive]`-decorated members.

## Build & Test Commands

```bash
dotnet restore
dotnet build --no-restore -c Release
dotnet test --no-build -c Release

# Run a single test project
dotnet test tests/ExpressiveSharp.Generator.Tests -c Release

# Run a single test by name
dotnet test --filter "FullyQualifiedName~MyTestName" -c Release

# Run tests against a specific target framework
dotnet test -f net8.0 -c Release

# Accept new Verify snapshots (for snapshot tests)
VERIFY_AUTO_APPROVE=true dotnet test tests/ExpressiveSharp.Generator.Tests

# Pack NuGet packages locally
dotnet pack -c Release

# Run benchmarks (all)
dotnet run -c Release --project benchmarks/ExpressiveSharp.Benchmarks/ExpressiveSharp.Benchmarks.csproj -- --filter "*"

# Run benchmarks (specific class)
dotnet run -c Release --project benchmarks/ExpressiveSharp.Benchmarks/ExpressiveSharp.Benchmarks.csproj -- --filter "*GeneratorBenchmarks*"
```

CI targets both .NET 8.0 and .NET 10.0 SDKs.

## Architecture

### Two-Phase Design

1. **Compile-time (source generation):** Two incremental generators in `ExpressiveSharp.Generator` (targets netstandard2.0):
   - `ExpressiveGenerator` — finds `[Expressive]` members, validates them via `ExpressiveInterpreter`, emits expression trees via `ExpressionTreeEmitter`, and builds a runtime registry via `ExpressionRegistryEmitter`
   - `PolyfillInterceptorGenerator` — uses C# 13 `[InterceptsLocation]` to rewrite `ExpressionPolyfill.Create()` and `IExpressiveQueryable<T>` LINQ call sites from delegate form to expression tree form. Supports all standard `Queryable` methods, multi-lambda methods (Join, GroupJoin, GroupBy overloads), non-lambda-first methods (Zip, ExceptBy, etc.), and custom target types via `[PolyfillTarget]` (e.g., EF Core's `EntityFrameworkQueryableExtensions` for async methods)

2. **Runtime:** `ExpressiveResolver` looks up generated expressions by (DeclaringType, MemberName, ParameterTypes). `ExpressiveReplacer` is an `ExpressionVisitor` that substitutes `[Expressive]` member accesses with the generated expression trees. Transformers (in `Transformers/`) post-process trees for provider compatibility.

### Key Source Files

- `src/ExpressiveSharp.Generator/ExpressiveGenerator.cs` — main generator entry point
- `src/ExpressiveSharp.Generator/Emitter/ExpressionTreeEmitter.cs` — maps IOperation nodes to `Expression.*` factory calls (the heart of code generation). Uses `varPrefix` to ensure unique local variable names across multi-lambda emitters
- `src/ExpressiveSharp.Generator/Interpretation/ExpressiveInterpreter.cs` — validates and prepares `[Expressive]` members
- `src/ExpressiveSharp.Generator/PolyfillInterceptorGenerator.cs` — interceptor generation. Dedicated emitters for complex methods (Join, GroupJoin, GroupBy multi-lambda), enhanced generic fallback (`EmitGenericSingleLambda`) for single-lambda methods with non-lambda arg forwarding, `[PolyfillTarget]` support for custom target types
- `src/ExpressiveSharp/Services/ExpressiveResolver.cs` — runtime expression registry lookup
- `src/ExpressiveSharp/PolyfillTargetAttribute.cs` — specifies a non-`Queryable` target type for interceptor forwarding (e.g., `EntityFrameworkQueryableExtensions`)
- `src/ExpressiveSharp/Extensions/ExpressiveQueryableLinqExtensions.cs` — delegate-based LINQ stubs on `IExpressiveQueryable<T>` (~85 intercepted + ~15 passthrough methods)
- `src/ExpressiveSharp.EntityFrameworkCore/Extensions/ExpressiveQueryableEfCoreExtensions.cs` — EF Core-specific stubs: chain-continuity (AsNoTracking, TagWith, etc.), Include/ThenInclude, and async lambda methods (AnyAsync, SumAsync, etc.)
- `src/ExpressiveSharp.EntityFrameworkCore/IIncludableExpressiveQueryable.cs` — hybrid interface bridging `IIncludableQueryable` and `IExpressiveQueryable` for Include/ThenInclude chain continuity
- `src/ExpressiveSharp.EntityFrameworkCore.RelationalExtensions.Abstractions/WindowFunction.cs` — public marker methods for all SQL window functions (ranking, aggregate, navigation). All throw at runtime; translated to SQL by the method call translator
- `src/ExpressiveSharp.EntityFrameworkCore.RelationalExtensions.Abstractions/WindowDefinition.cs` — fluent builder types: `PartitionedWindowDefinition` → `OrderedWindowDefinition` → `FramedWindowDefinition`. Type-safe chain ensures ORDER BY before ranking, frame only on aggregates/value functions
- `src/ExpressiveSharp.EntityFrameworkCore.RelationalExtensions.Abstractions/WindowFrameBound.cs` — frame boundary markers: `UnboundedPreceding` (property), `Preceding(n)` (method), `CurrentRow` (property), `Following(n)` (method), `UnboundedFollowing` (property)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EFNext/ExpressiveSharp](https://github.com/EFNext/ExpressiveSharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
