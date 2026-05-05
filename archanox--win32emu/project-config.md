---
trigger: always_on
description: Win32Emu is a Windows 32-bit PE executable emulator written in C# 14 (.NET 10) for running classic Windows games and applications on modern systems (Windows, Linux, macOS on x86 and ARM). It provides full x86 CPU emulation with JIT compilation, Windows API emulation (Kernel32, User32, DirectDraw, DirectInput, DirectSound, etc.), and cross-platform multimedia support through pluggable backends (SDL3, GLFW, Vulkan, Metal).
---

# GitHub Copilot Instructions for Win32Emu

## Project Overview

Win32Emu is a Windows 32-bit PE executable emulator written in C# 14 (.NET 10) for running classic Windows games and applications on modern systems (Windows, Linux, macOS on x86 and ARM). It provides full x86 CPU emulation with JIT compilation, Windows API emulation (Kernel32, User32, DirectDraw, DirectInput, DirectSound, etc.), and cross-platform multimedia support through pluggable backends (SDL3, GLFW, Vulkan, Metal).

## Coding Standards

### Language and Framework
- Use C# 14 with .NET 10 features
- Follow .editorconfig settings (tabs for indentation, tab width = 4)
- Use `var` for local variables when type is apparent or for built-in types
- Prefer block-scoped namespaces over file-scoped
- Use expression-bodied members for properties, indexers, and lambdas
- Avoid expression-bodied methods, constructors, and operators

### Logging
- **NEVER use `Console.WriteLine` for logging** - Use `ILogger` instance instead
- Example: `_logger.LogInformation("Message with {Parameter}", value);`
- Use structured logging with named parameters for better observability
- OpenTelemetry is integrated for logging, metrics, and tracing

### Naming Conventions
- Follow standard C# naming conventions (PascalCase for public members, camelCase for private fields)
- Use descriptive names that clearly indicate purpose
- For Win32 API emulation, match the original Win32 API naming (e.g., `CreateFileA`, `GetModuleHandleA`)

### Constants and Enums
- **Use enums instead of const for related constants** - Groups related values and provides type safety
- Example: Use `enum GdiObjectTypeId : uint { OBJ_PEN = 1, OBJ_BRUSH = 2 }` instead of separate const declarations
- For Win32 constants, prefer enums that match the underlying type (uint, int, etc.)
- **NEVER use magic numbers** - Always use named constants or enum values
  - Example: Use `(uint)NativeTypes.Win32Error.ERROR_MORE_DATA` instead of `234`
  - Example: Use `RPC_S_OK` constant instead of `0` in return statements

### Win32 API Module Functions
- **ALWAYS add `[DllModuleExport]` attribute** to all Win32 API function implementations
  - This attribute is **mandatory** for all public and private functions that are Win32 API exports
  - Include the export ordinal number: `[DllModuleExport(1)]`
  - For versioned exports, include version: `[DllModuleExport(241, Version = "4.90.0.3000")]`
  - **For stub implementations, add `IsStub = true`**: `[DllModuleExport(20, IsStub = true)]`
  - Example: `[DllModuleExport(20)] private uint WsprintfW(...)`

### Win32 Data Structures
- **Define Win32 structures as C# structs in NativeTypes.cs**
  - Do NOT use inline comments or typedef-style comments for structures used in implementation
  - Define the struct with proper field layout and offsets
  - Example:
    ```csharp
    public struct TIMECAPS
    {
        public uint wPeriodMin;  // Offset 0
        public uint wPeriodMax;  // Offset 4
    }
    ```
  - Use `System.Runtime.InteropServices.Marshal.SizeOf<T>()` to get structure size instead of hardcoding

### Code Organization
- Keep related functionality together in logical namespaces
- Use source generators for code generation tasks (see Win32Emu.Generators)
- Place emulated Win32 DLLs in their own modules (e.g., Win32Emu.Kernel32, Win32Emu.User32)

## Testing Requirements

### Test Structure
The project uses multiple test projects organized by component:
- `Win32Emu.Tests.Emulator` - CPU emulator conformance tests (required for CI)
- `Win32Emu.Tests.Kernel32` - Kernel32.dll API tests (optional for CI)
- `Win32Emu.Tests.User32` - User32.dll API tests (optional for CI)
- `Win32Emu.Tests.Gui` - GUI component tests
- `Win32Emu.Tests.CodeGen` - Code generation tests

### Test Guidelines
- **Core emulator tests are REQUIRED** - failures block PRs (CPU, memory, instruction execution)
- **Win32 DLL module tests are OPTIONAL** - failures don't block PRs (enables test-driven development)
- Each test should be independent and isolated
- Use descriptive test names that explain the scenario (e.g., `GetVersion_ReturnsWindows95_WhenVersionIs950`)
- Include both positive and negative test cases
- Use `TestEnvironment` class for consistent test setup
- Document any deviations from expected Win32 behavior with comments
- Mark tests with appropriate categories/traits

### Test Utilities
- Use `MockCpu` to simulate CPU without full emulation
- Use `TestEnvironment` for complete test setup (memory, CPU, process environment)
- Use memory helper utilities for string handling and memory access in tests

### Running Tests
```bash
dotnet test                              # All tests
dotnet test Win32Emu.Tests.Kernel32      # Specific project
dotnet test --filter "BasicFunctionsTests"  # Specific category
```

## Security Practices

### General Security
- Validate all external input (PE files, user input, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [archanox/Win32Emu](https://github.com/archanox/Win32Emu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
