---
trigger: always_on
description: CIL2CPP is a C# → C++ AOT compiler (similar to Unity IL2CPP).
---

# CLAUDE.md — CIL2CPP Project Instructions

## What is this project?

CIL2CPP is a C# → C++ AOT compiler (similar to Unity IL2CPP).
Pipeline: `.csproj` → `dotnet build` → .NET DLL (IL) → Mono.Cecil → IR → C++ source + CMakeLists.txt

Two halves:
- **Compiler** (`compiler/`): C# .NET 8 tool that reads IL and emits C++
- **Runtime** (`runtime/`): C++ 20 static library (BoehmGC, type system, BCL stubs)

## Project layout

```
compiler/
  CIL2CPP.CLI/          # Command-line entry point (System.CommandLine)
  CIL2CPP.Core/
    IL/                  # Mono.Cecil wrappers (AssemblyReader, TypeDefinitionInfo, ILInstruction)
    IR/                  # Intermediate representation (IRBuilder, IRModule, IRType, IRMethod, CppNameMapper)
    CodeGen/             # C++ code generator (CppCodeGenerator)
  CIL2CPP.Tests/         # xUnit tests (1,291 tests)
    Fixtures/            # SampleAssemblyFixture (builds sample DLLs once per run)
tests/                   # Test C# projects (compiler input): HelloWorld, ArrayTest, FeatureTest
runtime/
  include/cil2cpp/       # Public headers (types.h, object.h, string.h, array.h, gc.h, exception.h, boxing.h, type_info.h)
  src/                   # Implementation (gc/, exception/, type_system/, bcl/)
  tests/                 # Google Test (595 tests)
  cmake/                 # cil2cppConfig.cmake.in
tools/
  dev.py                 # Developer CLI (build/test/coverage/codegen/integration)
  integration_defs.py    # Data-driven integration test definitions (34 tests)
  integration_runner.py  # Parallel/sequential integration test executor
```

## Build commands

```bash
# Compiler (C#)
dotnet build compiler/CIL2CPP.CLI
dotnet build compiler/CIL2CPP.Core

# Runtime (C++)
cmake -B runtime/build -S runtime
cmake --build runtime/build --config Release
cmake --build runtime/build --config Debug
cmake --install runtime/build --config Release --prefix C:/cil2cpp
cmake --install runtime/build --config Debug --prefix C:/cil2cpp

# Code generation
dotnet run --project compiler/CIL2CPP.CLI -- codegen -i tests/HelloWorld/HelloWorld.csproj -o output
dotnet run --project compiler/CIL2CPP.CLI -- codegen -i tests/HelloWorld/HelloWorld.csproj -o output -c Debug

# One-step compile: .csproj → native executable
python tools/dev.py compile HelloWorld
python tools/dev.py compile HelloWorld --run         # compile and run
python tools/dev.py compile -i path/to/App.csproj    # arbitrary project
```

## Test commands

```bash
# C# unit tests (1,291 tests)
dotnet test compiler/CIL2CPP.Tests

# C# tests + coverage
dotnet test compiler/CIL2CPP.Tests --collect:"XPlat Code Coverage"

# C++ runtime tests (595 tests)
cmake -B runtime/tests/build -S runtime/tests
cmake --build runtime/tests/build --config Debug
ctest --test-dir runtime/tests/build -C Debug --output-on-failure

# Integration tests (204 tests across 34 projects) — full pipeline: csproj → codegen → cmake → build → run → verify
# Pre-builds CIL2CPP.CLI once, then workers use --no-build to avoid MSBuild lock contention
python tools/dev.py integration                    # parallel (auto-detect workers from CPU+RAM, ~6 min)
python tools/dev.py integration --sequential       # sequential (~21 min)
python tools/dev.py integration -j 2               # 2 parallel workers
python tools/dev.py integration --filter HelloWorld # run only matching tests

# All tests at once
python tools/dev.py test --all

# Coverage report (C# + C++ unified HTML)
python tools/dev.py test --coverage
```

## Architecture: Compiler pipeline

```
AssemblyReader (Mono.Cecil)
  → IRBuilder.Build() — 7 passes:
      Pass 1: Type shells (names, flags)
      Pass 2: Fields, base types, static constructors
      Pass 3: Method shells (signatures, no bodies)
      Pass 3.5: Generic method specializations (monomorphization)
      Pass 4: VTable construction
      Pass 5: Interface implementation mapping
      Pass 6: Method bodies (IL stack simulation → variable assignments)
      Pass 7: Record method synthesis (replace compiler-generated bodies)
  → CppCodeGenerator.Generate() — produces .h, .cpp, main.cpp, CMakeLists.txt
```

Key classes:
- `IRModule` contains `List<IRType>`, entry point, array init data, primitive type infos
- `IRType` contains fields, static fields, methods, VTable entries, base type ref
- `IRMethod` contains `List<IRBasicBlock>`, each block has `List<IRInstruction>`
- 25+ concrete `IRInstruction` subclasses (IRBinaryOp, IRCall, IRCast, IRBox, etc.)
- `CppNameMapper` maps IL type names ↔ C++ type names and mangles identifiers

## Coding conventions

### C# (compiler)
- .NET 8, C# latest, nullable enabled, implicit usings
- No .sln file — use `dotnet` commands directly with project paths
- xUnit for tests: `[Fact]`, `[Theory]`/`[InlineData]`, `[Collection("SampleAssembly")]`
- `SampleAssemblyFixture` builds sample DLLs once per test run via `ICollectionFixture<T>`
- **Tests MUST use fixture cache** (`GetXxxReleaseContext()` / `GetXxxReleaseModule()`) — never `new AssemblySet()` + `new ReachabilityAnalyzer()` in test methods (each costs ~12s)
- Mono.Cecil objects are owned by `AssemblyDefinition` — disposing `AssemblyReader` invalidates all Cecil-backed objects

### C++ (runtime)
- C++20, CMake 3.20+
- Headers in `include/cil2cpp/`, sources in `src/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kiroeko/cil2cpp](https://github.com/kiroeko/cil2cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
