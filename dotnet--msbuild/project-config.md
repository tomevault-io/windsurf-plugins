---
trigger: always_on
description: Instructions for GitHub Copilot and other AI coding agents working with the MSBuild repository.
---

# Agent Instructions

Instructions for GitHub Copilot and other AI coding agents working with the MSBuild repository.

## Repository Overview

**MSBuild** is the Microsoft Build Engine - performance-critical infrastructure for .NET and Visual Studio builds. This repository contains the code for the MSBuild build engine, including its public C# API, its internal implementation of the MSBuild programming language, and core targets and tasks used for builds.

### Key Components
- **Microsoft.Build**: Core MSBuild engine and public API
- **Microsoft.Build.Framework**: Framework interfaces and base types
- **Microsoft.Build.Tasks**: Built-in MSBuild tasks
- **Microsoft.Build.Utilities**: Utility classes for task authors
- **MSBuild CLI**: Command-line tool for invoking builds

### Technology Stack
- .NET 10.0 and .NET Framework 4.7.2
- C# 14 features (especially collection expressions)
- xUnit with Shouldly for testing
- Multi-platform support (Windows, Linux, macOS)

## General

* Performance is the top priority - minimize allocations, avoid LINQ in hot paths, use efficient algorithms.
* Always use the latest C# features, currently C# 14, especially collection expressions (`[]` over `new Type[]`).
* Match the style of surrounding code when making edits, but modernize aggressively for substantial changes.

## Code Review Instructions

Official builds treat all warnings as errors, so do not introduce new warnings.

### Performance Considerations

When reviewing pull requests:

* **Flag any unnecessary allocations** in hot paths
* **Flag LINQ usage** in performance-critical code paths
* **Check for proper use of `Span<T>`** and `ReadOnlySpan<T>` for string parsing
* **Ensure immutable collections** use the correct type (`ImmutableArray<T>` and `FrozenDictionary<TKey, TValue>` for read-heavy, `ImmutableList<T>` for incremental building)

### NuGet Feed Configuration

When reviewing pull requests:

* **Flag any changes to NuGet.config** that add external package sources without justification
* Package sources should use approved internal feeds when possible

## Formatting

* Apply code-formatting style defined in `.editorconfig`.
* Prefer file-scoped namespace declarations and single-line using directives.
* Insert a newline before the opening curly brace of any code block.
* Use pattern matching and switch expressions wherever possible.
* Use `nameof` instead of string literals when referring to member names.

### Nullable Reference Types

* **New files**: Always use nullable reference types (do NOT add `#nullable disable`)
* **Existing files with `#nullable disable`**: Match the existing style; don't add nullable annotations (`?`) to types
* **Existing files with nullable enabled**: Use proper nullable annotations
* Always use `is null` or `is not null` instead of `== null` or `!= null`

## Performance Best Practices

### Range Pattern Matching

```csharp
// GOOD: Clear and efficient
return errorNumber switch
{
    >= 3001 and <= 3999 => Category.Tasks,
    >= 4001 and <= 4099 => Category.General,
    >= 4100 and <= 4199 => Category.Evaluation,
    _ => Category.Other
};
```

### String Handling

* Use `MSBuildNameIgnoreCaseComparer` for case-insensitive comparisons of MSBuild names; use `StringComparer.OrdinalIgnoreCase` only for non-MSBuild string comparisons
* Use `char.ToUpperInvariant()` for single-character comparisons
* Use `ReadOnlySpan<char>` and `Slice()` to avoid string allocations
* Use `int.TryParse(span, out var result)` on .NET Core+ for allocation-free parsing

### Inlining Hot Paths

```csharp
[MethodImpl(MethodImplOptions.AggressiveInlining)]
private static bool IsCompilerPrefix(string value) => ...
```

### Immutable Collections

**Build once, read many times** (most common in MSBuild):
```csharp
ImmutableArray<string> items = source.Select(x => x.Name).ToImmutableArray();
FrozenDictionary<string, int> lookup = pairs.ToFrozenDictionary(x => x.Key, x => x.Value);
```

**Build incrementally over time**:
```csharp
ImmutableList<string> items = ...;  // Use when adding items one by one
ImmutableDictionary<string, int> lookup = ...;
```

## Building

NEVER pipe MSBuild output to a file. If you want a list of errors, add `-flp:"v=q;LogFile=ErrorsAndWarnings.log"` to the arguments.

When considering a subset of build errors, always look at the BEGINNING of the set, not the end.

## Individual projects

Build individual projects with `dotnet msbuild {path/to/project.csproj} -v:q`.

### Whole-repo Build Commands

| Platform | Command | Timeout |
|----------|---------|---------|
| Windows | `.\build.cmd -v quiet` | 300+ seconds (~2-3 minutes) |
| macOS/Linux | `./build.sh -v quiet` | 300+ seconds (~2-3 minutes) |

### Bootstrap Environment Setup

After building the whole repo, to use the just-built MSBuild to build things use the bootstrap environment (described in [`skills/use-bootstrap-msbuild](.github/skills/use-bootstrap-msbuild/SKILL.md)).

### Build Troubleshooting

* If build fails with "Could not resolve SDK", run the bootstrap environment script
* Verify `dotnet --version` shows the preview/internal version
* Use repository sample projects for testing, not external projects
* Build artifacts go to `./artifacts/` directory

### Running Tests


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dotnet/msbuild](https://github.com/dotnet/msbuild) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
