---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A [Fody](https://github.com/Fody/Fody) weaver. Users derive aspects from `OnMethodBoundaryAspect` (overriding `OnEntry`/`OnExit`/`OnException`) and apply them to methods, classes or the assembly; at build time the weaver rewrites the IL of the target methods to call the aspects. The NuGet package `MethodBoundaryAspect.Fody` contains both the runtime attributes and the weaver.

## Commands

The CI (`.github/workflows/main.yaml`, Windows) does exactly this:

```bash
dotnet build --configuration Release src/MethodBoundaryAspect.Fody.sln
dotnet test src/MethodBoundaryAspect.Fody.UnitTests.NetFramework --configuration Release --no-build
dotnet test src/MethodBoundaryAspect.Fody.UnitTests.NetCore --configuration Release --no-build
dotnet test src/MethodBoundaryAspect.Fody.RuntimeTests --configuration Release --no-build
```

- Single test: `dotnet test <project> -c Release --no-build --filter "FullyQualifiedName~ClassName.MethodName"`; RuntimeTests on one runtime: add `-f net8.0`.
- Build with `-m:1 -nodeReuse:false`: a parallel build can fail with a locked `TestAssembly.NetFramework.dll` (it is built by two projects), and a reused MSBuild node may still hold a weaver DLL from another checkout ("Assembly with same name is already loaded").
- RuntimeTests uses the weaver DLL from `src/MethodBoundaryAspect.Fody/bin/<Configuration>` (no project reference), so building only RuntimeTests does not rebuild the weaver, and an incremental build does not re-weave. After changing the weaver, build `src/MethodBoundaryAspect.Fody` (or the solution) first, then RuntimeTests with `--no-incremental`.
- **Path length:** the NetFramework tests fail (PEVerify: "file name too long") and can hang when run from a deep path such as `.claude/worktrees/<name>`. Run them from a short path, e.g. `subst Q: <repo>` and `dotnet test` from `Q:\`, then `subst Q: /d`.
- Benchmark (weaves with the local weaver): `dotnet run -c Release --project src/MethodBoundaryAspect.Fody.Benchmark -- --filter '*' --job short`
- `Directory.Build.props`: C# 8, `TreatWarningsAsErrors`, strong-name signing.
- Versioning is GitVersion (default config): `+semver: major|minor` in a commit message bumps the version. Pushing a `v*` tag publishes to nuget.org.

## Projects

- `src/MethodBoundaryAspect` – runtime library and NuGet package (`OnMethodBoundaryAspect`, `MethodExecutionArgs`, `FlowBehavior`, ordering/filter attributes). Packs the weaver via FodyPackaging.
- `src/MethodBoundaryAspect.Fody` – the weaver. **netstandard2.0 only**, because Fody only loads that binary. Uses Mono.Cecil.
- `UnitTests.NetFramework` (net462, largest suite) – weaves `UnitTests.TestAssembly.NetFramework` (and a VB and an unverifiable test assembly) into a shadow copy `_<tick>_..._Weaved_.dll`, runs PEVerify, loads it into a separate AppDomain and invokes methods via `AssemblyLoader`. Base classes: `MethodBoundaryAspectTestBase` (Shared), `MethodBoundaryAspectNetFrameworkTestBase`.
- `UnitTests.NetCore` (net8) – weaves `UnitTests.TestAssembly.NetCore`/`NetStandard` and a net462 copy, inspects the woven IL with Cecil or captures warnings; tests that weave the same assembly share an xUnit collection because shadow files are named by `Environment.TickCount`.
- `RuntimeTests` (net462, net48, net8.0, net9.0, net10.0) – real Fody build with the local weaver (`<WeaverFiles ... WeaverClassNames="ModuleWeaver" />`), then plain xUnit tests of the woven code on each runtime. Its `FodyWeavers.xml` element is therefore `<ModuleWeaver>`; NuGet users write `<MethodBoundaryAspect>`. Prefer this project for behavior tests of woven code.

## Weaver architecture

`ModuleWeaver.Execute` (reads `FodyWeavers.xml` config, e.g. `SuppressRefStructWarnings`, `DisableExecutionArgsOptimization`) walks all types/nested types, collects aspects from assembly, class and method (property aspects from the property) and filters them (`[DisableWeaving]`, visibility via `AttributeTargetMemberAttributes`, Namespace/TypeName/MethodName regex filters, self-weaving). `AspectOrderer` sorts them (`[AspectOrderIndex]`, `[ProvideAspectRole]`/`[AspectRoleDependency]`).

`MethodWeaverFactory` drops aspects that override none of the three methods and picks:
- `MethodWeaver` (sync): moves the original body into a new private `$_executor_<Name>` method (`AggressiveInlining`) and rewrites the original method as: create arguments array → `MethodExecutionArgs` → aspect instances (a new instance **per call**, ctor args/properties/fields replayed from the attribute) → `OnEntry` per aspect with `FlowBehavior` checks → call executor → `OnExit` in reverse order → return; a try/catch with `OnException` is only emitted if an aspect overrides it. With multiple aspects each aspect's `MethodExecutionTag` is saved/restored around its calls.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vescon/MethodBoundaryAspect.Fody](https://github.com/vescon/MethodBoundaryAspect.Fody) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
