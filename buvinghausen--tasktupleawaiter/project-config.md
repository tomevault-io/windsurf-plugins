---
trigger: always_on
description: TaskTupleAwaiter provides extension methods that allow you to `await` a `ValueTuple` of `Task<T>` (or non-generic `Task`) instances and destructure the results in a single line. In this repository, a Roslyn incremental source generator (`src/TaskTupleAwaiter.Generator`) generates the extension-method source during library build under `namespace System.Threading.Tasks`, and that generated code is compiled into `TaskTupleAwaiter.dll` for each target framework. Consumers install and reference the c
---

# CLAUDE.md — TaskTupleAwaiter

## Project Overview

TaskTupleAwaiter provides extension methods that allow you to `await` a `ValueTuple` of `Task<T>` (or non-generic `Task`) instances and destructure the results in a single line. In this repository, a Roslyn incremental source generator (`src/TaskTupleAwaiter.Generator`) generates the extension-method source during library build under `namespace System.Threading.Tasks`, and that generated code is compiled into `TaskTupleAwaiter.dll` for each target framework. Consumers install and reference the compiled package binaries; the generator is a private build-time implementation detail.

## Development Workflow — Use Superpowers

This repo has the `superpowers` plugin enabled (`.claude/settings.json`). It's not decorative —
use the skill tree for real work here, and it maps directly onto the spec-first, plan-second,
code-last workflow:

| Phase | Skill |
|---|---|
| Design/spec iteration | `brainstorming` |
| Plan once the spec settles | `writing-plans` |
| Implementation | `executing-plans` (pairs with `test-driven-development`) |
| Bug fixes (spec-first exception) | `systematic-debugging` |
| Before calling anything done | `verification-before-completion` |
| Opening/handling a PR | `requesting-code-review` / `receiving-code-review` |
| Wrapping up a branch | `finishing-a-development-branch` |

If a skill applies to what you're doing, invoke it — don't just read the table and proceed
manually. The transition points (spec → plan, plan → code) are still explicit human decisions,
per the global CLAUDE.md; the skills are how each phase gets executed, not a way around the
hand-off.

## Repository Layout

```
TaskTupleAwaiter/
├── src/
│   ├── TaskTupleAwaiter/                  # Main library shell (netstandard2.0, net462, net8.0, net9.0)
│   │                                      #   No hand-authored .cs sources — code is generated at build and compiled into the library.
│   └── TaskTupleAwaiter.Generator/        # Roslyn incremental source generator (netstandard2.0)
│       └── TaskTupleExtensionsGenerator.cs
├── tests/
│   ├── unit/
│   │   └── TaskTupleAwaiter.Tests/        # xUnit v3 test project (net11.0, net10.0, net9.0, net8.0, net472; runtime-async=on for net11.0)
│   │       ├── TaskTupleAwaiterTests.cs
│   │       ├── BehaviorComparisonTests.cs
│   │       ├── Adapters/                  #   AwaiterAdapter partial classes
│   │       ├── DummyException.cs
│   │       ├── On.cs
│   │       ├── CopyableSynchronizationContext.cs
│   │       └── SpySynchronizationContext.cs
│   └── smoke/
│       └── TaskTupleAwaiter.AotSmokeTest/ # NativeAOT downstream-consumer smoke-test (net8.0, net9.0, net11.0)
│           ├── TaskTupleAwaiter.AotSmokeTest.csproj
│           └── Program.cs
├── benches/
│   └── TaskTupleAwaiter.Benchmarks/       # BenchmarkDotNet harness (net8.0, net9.0)
│       ├── TaskTupleAwaiter.Benchmarks.csproj  # Sibling of tests/, so it doesn't inherit xUnit/Shouldly from tests/unit/Directory.Build.props.
│       ├── Program.cs                     #   BenchmarkSwitcher entry point.
│       ├── TypedTupleAwaitBenchmarks.cs
│       ├── NonGenericTupleAwaitBenchmarks.cs
│       ├── ConfigureAwaitBenchmarks.cs
│       └── README.md                      #   How to run; the runs are local-only, not CI.
├── docs/superpowers/                      # Specs and implementation plans
├── README.md
├── LICENSE.txt
├── test.sh                                # Linux/WSL2 test runner — see Build & Test below
└── CLAUDE.md                              # This file
```

## Technology Stack

| Concern | Choice |
|---|---|
| Language | C# 14.0 |
| Library TFMs | netstandard2.0, net462, net8.0, net9.0 |
| Generator target | netstandard2.0 (Roslyn analyzer requirement) |
| AOT-compatible TFMs | net8.0+ (`<IsAotCompatible>true</IsAotCompatible>` via `IsTargetFrameworkCompatible`) |
| Generator framework | Roslyn `IIncrementalGenerator` |
| Test framework | xUnit v3 |
| Assertion library | Shouldly |
| Max tuple arity | 16 |

## Key Design Decisions

### Source Generator (`TaskTupleExtensionsGenerator`)
- Implements `IIncrementalGenerator` (not the older `ISourceGenerator`).
- **Feature-detects** `ConfigureAwaitOptions` at compile time by resolving the type `System.Threading.Tasks.ConfigureAwaitOptions` from the target compilation — **do not use** `#if NET8_0_OR_GREATER` or preprocessor symbols.
- Emits `Task.WhenAll([tasks.Item1, ..., tasks.ItemN])`. Overload binding is determined by the library TFM: `netstandard2.0` / `net462` / `net8.0` bind to `Task.WhenAll(params Task[])` (heap-allocated array — same IL as before), while `net9.0`+ binds to `Task.WhenAll(ReadOnlySpan<Task>)` and stack-allocates the buffer. No runtime feature detection needed for this — overload preference is purely a compiler/TFM concern.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [buvinghausen/TaskTupleAwaiter](https://github.com/buvinghausen/TaskTupleAwaiter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
