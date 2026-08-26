---
trigger: always_on
description: Keysharp is a cross-platform C# implementation of AutoHotkey v2. AHK scripts (`.ahk`/`.ks`) are parsed by a hand-written lexer + recursive-descent parser into an AST, lowered to C# (Roslyn `SyntaxNode`s), compiled in-memory with Roslyn, and executed as a .NET assembly. The parser and compiler are optional first-party deployment units, so compiled scripts can run with the runtime alone. The goal is full AHK v2 compatibility on Windows, with partial Linux and eventual macOS support.
---

# Keysharp — Claude Code Guide

## Project overview

Keysharp is a cross-platform C# implementation of AutoHotkey v2. AHK scripts (`.ahk`/`.ks`) are parsed by a hand-written lexer + recursive-descent parser into an AST, lowered to C# (Roslyn `SyntaxNode`s), compiled in-memory with Roslyn, and executed as a .NET assembly. The parser and compiler are optional first-party deployment units, so compiled scripts can run with the runtime alone. The goal is full AHK v2 compatibility on Windows, with partial Linux and eventual macOS support.

## Solution structure

```
Keysharp.sln
├── Keysharp/               # Entry-point executable (thin launcher)
├── Keysharp.Core/          # Runtime and built-in logic; no parser or Roslyn dependency
│   ├── Builtins/           # AHK built-in functions (Keyboard, GUI, Files, COM, …)
│   ├── Internals/          # Platform services: hooks, threading, window, input
│   │   ├── Input/Hooks/    # HookThread (4 k lines) — OS keyboard/mouse callbacks
│   │   ├── Input/Keyboard/ # HotkeyDefinition, KeyboardMouseSender
│   │   └── Threading/      # Threads, ScriptTimerManager, SlimStack, ThreadVariables
│   └── Runtime/Script/     # Script singleton, Call helpers, event scheduler
├── Keysharp.Components/    # Independently deployable first-party components
│   ├── Scripting/
│   │   ├── Contracts/      # Small Roslyn-free component contract
│   │   ├── Parser/         # Lexer/parser and syntax validation; no Roslyn
│   │   └── Compiler/       # Lowerer and Roslyn compiler
│   └── Packages/
│       ├── Contracts/      # Keysharp.Components.Packages provider contract
│       └── NuGet/          # In-process NuGet package-provider implementation
├── Keysharp.Tests/         # NUnit test suite; test scripts in Code/
├── Keysharp.Benchmark/     # BenchmarkDotNet perf suite
├── Keysharp.OutputTest/    # Compiled-exe smoke tests (Program.cs is generated output)
└── Keyview/                # Script viewer tool (Scintilla-based in Windows, Eto.Forms-based in other platforms)
```

## Build

Target framework is **net10.0** (net10.0-windows on Windows). Platform is **AnyCPU** everywhere, on every OS.

Nothing needs a machine type in the managed assemblies: native dependencies are resolved at runtime through
the RID graph in `deps.json`, the x64-only DllCall register shim is guarded by a `ProcessArchitecture` check,
and scripts are Roslyn-compiled as AnyCpu. So architecture is chosen with **`-r <rid>`**, never `-p:Platform`
— a RID-less `dotnet build` simply takes the apphost matching the SDK doing the building, which is what makes
the same checkout work on x64 and on Windows on ARM.

```bash
dotnet build Keysharp.sln -c Debug                       # runnable on this machine, whatever its arch
dotnet publish Keysharp/Keysharp.csproj -c Release -r win-x64
dotnet publish Keysharp/Keysharp.csproj -c Release -r win-arm64
```

Scripts can branch on architecture with the predefined `X64` / `ARM64` / `X86` / `ARM` preprocessor symbols,
or at runtime via `#Import Ks { A_ProcessArch, A_OSArch }`. Use those rather than `A_PtrSize`, which is 8 for
both x64 and ARM64.

```bash
# Windows — from repo root
dotnet build Keysharp.sln -c Debug

# Linux — requires sibling Eto fork cloned at ../Eto
# git clone -b Keysharp https://github.com/keysharp-org/Eto.git ../Eto
dotnet build Keysharp.sln -c Debug
```

Build output lands in `bin/Debug/net10.0-windows/` (or the appropriate TFM subfolder) due to `Directory.Build.props`.

## Run a script

```bash
# Windows
./bin/Debug/net10.0-windows/Keysharp.exe myscript.ahk

# Useful flags
--transpile          # emit generated C# alongside the script and exit (great for debugging transpiler)
--validate           # compile-check only, do not run
--compile exe        # emit a standalone .exe and exit
--compile exe-min    # like exe, with dependencies embedded
--compile asm        # emit a .cks assembly beside the script and exit; several scripts may follow,
                     #   each compiled beside its own source, sharing one warm compiler
--dest <path>        # send --compile's output to <path> (or * for stdout); one output, so one script
```

> **Important**: a script an agent runs must never be able to block on a dialog unless the user has explicitly opted in. A load-time warning or an uncaught runtime error opens a modal window on the user's desktop that nothing in this environment can dismiss, and it leaves an orphaned `Keysharp.exe` behind. Before running a script:
>
> - Put `#ErrorStdOut` at the top of it. The directive sends any syntax error that prevents the script from launching — and any uncaught runtime error — to the standard error stream (stderr) rather than displaying a dialog.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keysharp-org/Keysharp](https://github.com/keysharp-org/Keysharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
