---
trigger: always_on
description: This repository contains **Nethermind.Int256**, a high-performance .NET library implementing 256-bit integer types for blockchain and cryptographic applications. The library provides both signed (`Int256`) and unsigned (`UInt256`) 256-bit integer implementations optimized for performance using hardware intrinsics and vectorization.
---

# GitHub Copilot Instructions for Int256 Repository

## Project Overview

This repository contains **Nethermind.Int256**, a high-performance .NET library implementing 256-bit integer types for blockchain and cryptographic applications. The library provides both signed (`Int256`) and unsigned (`UInt256`) 256-bit integer implementations optimized for performance using hardware intrinsics and vectorization.

### Key Features
- **High Performance**: Leverages SIMD instructions, vectorization, and hardware intrinsics
- **Complete API**: Implements all standard arithmetic, bitwise, and comparison operations
- **.NET Integration**: C# with .NET 9.0 target framework, full compatibility with .NET numeric interfaces and conversion patterns
- **Cross-Platform**: Supports multiple architectures with optimized code paths
- **Memory Efficient**: Struct-based design with minimal allocation overhead


## General

- Make only high confidence suggestions when reviewing code changes.
- Always use the latest version C#, currently C# 13 features.
- Never change global.json unless explicitly asked to.
- Never change package.json or package-lock.json files unless explicitly asked to.
- Never change NuGet.config files unless explicitly asked to.
- Always trim trailing whitespace, and do not have whitespace on otherwise empty lines.

**Any code you commit SHOULD compile, and new and existing tests related to the change SHOULD pass.**

You MUST make your best effort to ensure your changes satisfy those criteria before committing. If for any reason you were unable to build or test the changes, you MUST report that. You MUST NOT claim success unless all builds and tests pass as described above.

You MUST follow all code-formatting and naming conventions defined in [`.editorconfig`](/.editorconfig).

In addition to the rules enforced by `.editorconfig`, you SHOULD:

- Prefer file-scoped namespace declarations and single-line using directives; however do not change the type of namespace format in an existing file unless specifically asked.
- Ensure that the final return statement of a method is on its own line.
- Use pattern matching and switch expressions wherever possible.
- Use `nameof` instead of string literals when referring to member names.
- Always use `is null` or `is not null` instead of `== null` or `!= null`.
- Trust the C# null annotations and don't add null checks when the type system says a value cannot be null.
- Prefer `?.` if applicable (e.g. `scope?.Dispose()`).
- Use `ObjectDisposedException.ThrowIf` where applicable.
- When adding new unit tests, strongly prefer to add them to existing test code files rather than creating new code files.
- If you add new code files, ensure they are listed in the csproj file (if other files in that folder are listed there) so they build.
- When running tests, if possible use filters and check test run counts, or look at test logs, to ensure they actually ran.
- Do not finish work with any tests commented out or disabled that were not previously commented out or disabled.
- When writing tests, do not emit "Act", "Arrange" or "Assert" comments.
- Copy existing style in nearby files for test method names and capitalization.
- Provide code comments when helpful to explain why something is being done; however do not comment what is obvious and just a repeation of the code line.
- Ensure that XML doc comments are created for any public APIs.
- Do NOT use #regions.
- Prefer low allocation and higher performance code.

---


## Architecture and Core Components

### Core Types
- **`UInt256`**: 256-bit unsigned integer (primary implementation)
- **`Int256`**: 256-bit signed integer (wrapper around UInt256)
- **`IInteger<T>`**: Common interface for integer operations
- **`BigIntegerExtensions`**: Extensions for System.Numerics.BigInteger integration

### Internal Structure
```csharp
// UInt256 uses explicit layout with 4 ulong components
[StructLayout(LayoutKind.Explicit)]
public readonly struct UInt256
{
    [FieldOffset(0)] public readonly ulong u0;  // Least significant
    [FieldOffset(8)] public readonly ulong u1;
    [FieldOffset(16)] public readonly ulong u2;
    [FieldOffset(24)] public readonly ulong u3; // Most significant
}
```

### Performance Optimizations
- **Hardware Intrinsics**: Uses `Vector256<T>` when available
- **Conditional Compilation**: Hardware intrinsics can be disabled via `DOTNET_EnableHWIntrinsic=0`
- **Unsafe Operations**: Leverages unsafe code for optimal performance
- **Branch Optimization**: Minimizes conditional branches in hot paths

## Development Environment

### Requirements
- **.NET 9.0 SDK** (specified in global.json)
- **Visual Studio 2022** or **VS Code** with C# extension
- **Git** for version control

### Project Structure
```
src/
├── Nethermind.Int256/              # Core library
│   ├── Int256.cs                   # Signed 256-bit integer
│   ├── UInt256.cs                  # Unsigned 256-bit integer (main implementation)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NethermindEth/int256](https://github.com/NethermindEth/int256) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
