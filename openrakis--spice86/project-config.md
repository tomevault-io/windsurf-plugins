---
trigger: always_on
description: Guidance for AI agents working in the Spice86 repository.
---

# AGENTS

Guidance for AI agents working in the Spice86 repository.

## Project Overview
Spice86 is a .NET 10 cross-platform emulator for reverse engineering real-mode DOS programs. It enables running, analyzing, and incrementally rewriting DOS binaries in C# without source code.

## Architecture & Module Boundaries

### Project Structure
- **`Spice86.Core`**: Core emulation engine (CPU, memory, devices, DOS/BIOS handlers)
- **`Spice86`**: Main application with Avalonia UI (ViewModels, Views, manual composition root)
- **`Bufdio.Spice86`**: Audio subsystem (PortAudio bindings)
- **`Spice86.Logging`**: Serilog-based logging infrastructure
- **`Spice86.Shared`**: Shared interfaces and utilities
- **`Spice86.Tests`**: XUnit tests with FluentAssertions and NSubstitute

### Dependency Injection
The entire emulator is assembled in `Spice86DependencyInjection.cs` (~600 lines):
- Constructor creates the full object graph with explicit dependencies (no IoC container)
- Order matters: components are constructed in dependency order
- Components are wired together with event handlers and shared state
- Entry point is `Program.cs` which instantiates `Spice86DependencyInjection`
- **`Spice86DependencyInjection` is the central composition root** - understand its structure when working with dependencies
- The `Machine` class aggregates emulator components (CPU, memory, devices) - access via properties like `CfgCpu`, `Memory`, `Stack`
- `InterruptVectorTable` and `Stack` are now passed directly to `Machine` constructor

### CPU Execution Model
**`CfgCpu`** is the sole CPU implementation (Control Flow Graph-based executor):
- Builds dynamic CFG for analysis and future JIT compilation
- Tracks instruction variants for self-modifying code via selector nodes
- Maintains execution context hierarchy for hardware interrupts
- See `doc/cfgcpuReadme.md` for detailed CFG architecture

## Critical Workflows

### Building & Running
```powershell
# Build from solution root
dotnet build

# Run with executable
dotnet run --project src/Spice86 -- -e path\to\program.exe

# Run tests (excluding SingleStepTest - see note below)
dotnet test tests/Spice86.Tests --filter 'FullyQualifiedName!~SingleStepTest'
```

> **SingleStepTest exclusion rule**: `SingleStepTest` runs millions of CPU instruction test cases and take an extremely long time to complete. **Always exclude it** using `--filter 'FullyQualifiedName!~SingleStepTest'` unless the change being tested directly touches CPU instruction decoding, execution, or flag handling (e.g. changes to `CfgCpu`, instruction parsers, ALU operations, or flag computation). When in doubt, exclude it.

> **Test output rule (agents only)**: no need to redirect `dotnet test` output to a file. On success the console summary is short (a single `Passed! - Failed: 0, Passed: N ...` line), so run it directly and read the summary. Only capture to a file when a run actually fails and you need the full failure detail.

> **Long-running test rule (Kiro agents only)**: the foreground shell tool (`execute_bash`) enforces a hard wall-clock cap (roughly 25-60s in practice) that fires *before* the `timeout` argument. When it trips it force-returns exit 1 while the detached `dotnet` process keeps running and completes normally, which looks exactly like a crash (abrupt exit 1, no test summary, no dump) but is not one. Any test run expected to exceed ~25s (the full suite is ~2 min; `CfgGraphReloadTest` alone is ~55s) MUST be launched with the background-process tool (which has no such cap), writing a completion marker, then polled with short (<20s) foreground calls until the marker appears. Do not conclude a suite "crashed" from a bare exit 1 with no summary; re-run it in the background and check the real result first. Splitting `--filter` so each foreground run stays under the cap also works.

### Debugging Workflow
- **GDB Integration**: Server runs on port 10000 by default (`--GdbPort 10000`)
  - Use `--Debug` to pause at startup for breakpoint setup
  - Custom GDB commands via `monitor` (e.g., `monitor dumpall`, `monitor breakCycles 1000`)
- **Seer Client**: Use `seergdb --project doc/spice86.seer` for GUI debugging
- **Internal Debugger**: UI-based debugger with disassembly, memory, CPU state views

### Reverse Engineering Process
1. Run DOS program in Spice86
2. Emulator dumps `spice86dumpMemoryDump.bin` and `spice86dumpExecutionFlow.json` to `--RecordedDataDirectory`
3. Load dumps in Ghidra via [spice86-ghidra-plugin](https://github.com/OpenRakis/spice86-ghidra-plugin)
4. Generate C# override classes from decompiled functions
5. Implement `IOverrideSupplier` to register overrides at segmented addresses
6. Run with `--UseCodeOverride true` to replace assembly with C# incrementally

## Code Override System

### Override Registration Pattern
```csharp
public class MyOverrideSupplier : IOverrideSupplier {
    public IDictionary<SegmentedAddress, FunctionInformation> GenerateFunctionInformations(
        ILoggerService loggerService, Configuration configuration, 
        ushort programStartSegment, Machine machine) {
        return new MyOverrides(new(), machine, loggerService, configuration).FunctionInformations;
    }
}

public class MyOverrides : CSharpOverrideHelper {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenRakis/Spice86](https://github.com/OpenRakis/Spice86) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
