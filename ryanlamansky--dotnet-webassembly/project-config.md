---
trigger: always_on
description: Guidance for working in this repository.
---

# CLAUDE.md

Guidance for working in this repository.
Keep it accurate — update it when the layout, commands, or conventions change.

Personal, machine-specific preferences can go in `CLAUDE.local.md` (gitignored, optional) — it's imported here:

@CLAUDE.local.md

## What this project is

**WebAssembly for .NET** — a pure-.NET library (NuGet package `WebAssembly`) to create, read, modify, write, and execute WebAssembly (`.wasm`) binaries, plus convert them to .NET DLLs.
Execution is not an interpreter: WASM instructions are emitted as .NET IL via `System.Reflection.Emit` and JIT-compiled to native code by the CLR.

- Execution targets ratified WASM **3.0** (released 2025-09-17) — extended constant expressions, tail calls, multiple memories, 64-bit address space (memory64 / table64), typed function references, exception handling (the `try_table` model), relaxed SIMD (deterministic profile), and garbage collection (struct / array / i31) — on top of the earlier 2.0 feature set (bulk memory, reference types, fixed-width SIMD, multi-value, typed `select`).
  Spec compliance is strong: every ratified-3.0 spec-suite category runs green.
  The only intentional exclusions are the seven `float_exprs` NaN-folding lines marked `unsupported` in `SpecTests.cs` (the JIT folds `x±0` / `x*1` so a signaling NaN is never quieted), and they keep the category green rather than deferring it.
  This library targets the **deterministic profile**: float operations follow IEEE-754 via the CLR, relaxed-SIMD instructions implement the spec's deterministic defaults, and NaN payload bit patterns are the only known divergence.
  Post-3.0 proposals (custom page sizes, wide arithmetic, JS string builtins, …) aren't covered.
- `WebAssembly.Module` is the object-model root for reading, writing, and modifying.
  It exposes typed section collections — `Types`, `Imports`, `Functions`, `Tables`, `Memories`, `Globals`, `Exports`, `Elements`, `Codes`, `Data`, `CustomSections` — with `ReadFromBinary()` / `WriteToBinary()` for serialization.
- `WebAssembly.Runtime.Compile` drives execution and the WASM→DLL path.
  `Compile.FromBinary<TExports>(...)` takes an abstract class whose members map to WASM exports plus an `ImportDictionary`, and returns an `InstanceCreator<TExports>` factory.
  `Compile.CreatePersistedAssembly(...)` is the .NET 9+ path that emits a DLL instead (via `PersistedAssemblyBuilder`); it shares the compiler with in-memory execution, and the full spec suite runs green through it (`PersistedSpecTests.cs`).
- Imports are supplied through `ImportDictionary`, keyed by module/field name: `FunctionImport` (wraps a delegate), `MemoryImport`, `GlobalImport`, `TableImport`.

## Layout

| Path | What it holds |
|------|---------------|
| `WebAssembly/` | The library. Top-level types are the WASM object model (`Module.cs`, `Function.cs`, `Export.cs`, `OpCode.cs`, …). |
| `WebAssembly/Instructions/` | One file per instruction, each a subclass of `Instruction`. |
| `WebAssembly/Runtime/` | Execution: `Compile.cs`, configuration, runtime exceptions, import types, `UnmanagedMemory`. |
| `WebAssembly/Runtime/Compilation/` | The IL-emission engine: `CompilationContext`, `BlockStack`, `Signature`, `HelperMethod`, IL extensions. |
| `WebAssembly.Tests/` | MSTest test project. Mirrors the library: `Instructions/`, `Runtime/`. |
| `WebAssembly.Tests/Runtime/SpecTestData/` | **Generated** spec-suite fixtures (`.wasm` + `.json`). Do not hand-edit. |
| `Tools/RefreshSpecTests/` | Tool that regenerates `SpecTestData/` from the upstream WebAssembly spec suite. |
| `Examples/` | Standalone runnable samples (`RunExisting`, `GenerateClassFromWasm`, `ReadMeSample`). |
| `docs/` | `BreakingChanges.md`, the breaking-change log. |

## Build & test

The solution `WebAssembly.sln` is the build root.
Requires the .NET 8, 9, and 10 SDKs.

```bash
dotnet build                       # build everything (Debug)
dotnet build -c Release            # Release can differ — see note below
dotnet test                        # run the full MSTest suite
dotnet test -c Release
dotnet test --filter "FullyQualifiedName~SpecTest_address"   # one test / class
```

CI (`.github/workflows/dotnetcore.yml`) builds **and** tests in **both Debug and Release**, because conditional compilation makes them genuinely different.
If you change anything platform- or `#if`-conditional, validate both configurations locally before assuming green.

Target frameworks:
- Library `WebAssembly.csproj`: `net8.0;net9.0` (net8.0 is the oldest supported baseline; .NET Standard 2.0 was dropped in 2.0.0; net8/net9 both leave support around Nov 2026).
  A net10.0 target is added only when the code actually uses a .NET 10 API behind `#if NET10_0_OR_GREATER` — an extra target with no conditional code is identical IL and pure package weight (net10 runtimes get the net9 asset and all the JIT uplift regardless).
- Tests: `net8.0;net9.0;net10.0` (the net10 cell exercises the newest-asset-on-newest-runtime pairing users actually get).

## Conventions

- `TreatWarningsAsErrors=True` on the library and tests.
  A warning fails the build.
  `AnalysisMode=Recommended` with `EnforceCodeStyleInBuild=true`.
- `Nullable` is enabled everywhere.
  Honor nullability; don't paper over it with `!` unless there's a real invariant.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RyanLamansky/dotnet-webassembly](https://github.com/RyanLamansky/dotnet-webassembly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
