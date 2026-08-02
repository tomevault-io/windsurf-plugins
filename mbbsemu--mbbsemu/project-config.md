---
trigger: always_on
description: This directory contains `ExecutionUnit.cs`, a single but critical class that encapsulates everything needed to run a chunk of x86 code within a module: a CPU core, registers, memory reference, exported module dictionary, and interrupt handlers. ExecutionUnits are the mechanism that enables nested subroutine calls without corrupting CPU state.
---

# ExecutionUnits — CPU Execution Contexts

This directory contains `ExecutionUnit.cs`, a single but critical class that encapsulates everything needed to run a chunk of x86 code within a module: a CPU core, registers, memory reference, exported module dictionary, and interrupt handlers. ExecutionUnits are the mechanism that enables nested subroutine calls without corrupting CPU state.

## Why ExecutionUnits Exist

The original MajorBBS was single-threaded, but its API allows re-entrant calls. For example, while EU0 is executing a module's main handler, that handler might call `outprf()`, which calls `ProcessTextVariables()`, which needs to execute a module-registered text variable callback — a completely separate x86 subroutine. That callback needs its own CPU state (registers, instruction pointer) but must share the same module memory.

**The solution**: Each `MbbsModule` maintains a pool (`Queue<ExecutionUnit>`) of ExecutionUnits. They all share the same `IMemoryCore` (including stack segment), but each has its own `CpuCore` with independent registers. When a nested call is needed, a new EU is dequeued (or created on demand), executes with isolated register state, and is returned to the pool.

```
MbbsHost.Run()
  └─ MbbsModule.Execute()           ← dequeues EU0
       └─ EU0.Execute()             ← runs x86 code
            └─ CALL FAR 0xFFFF:326  ← hits ExternalFunctionDelegate
                 └─ Majorbbs.outprf()
                      └─ ProcessTextVariables()
                           └─ Module.Execute()  ← dequeues EU1 (nested!)
                                └─ EU1.Execute() ← runs text var callback
                                     └─ returns, EU1 re-enqueued
                      └─ outprf() continues
            └─ EU0 continues with its registers intact
       └─ EU0 re-enqueued
```

## ExecutionUnit.cs — Anatomy

### Fields

```csharp
public readonly ICpuCore ModuleCpu;                              // Own CpuCore instance
public readonly ICpuRegisters ModuleCpuRegisters;                // = ModuleCpu (CpuCore implements ICpuRegisters)
public readonly IMemoryCore ModuleMemory;                        // SHARED with module and other EUs
public readonly Dictionary<ushort, IExportedModule> ExportedModuleDictionary;  // SHARED
public string Path { get; init; }                                // Module file path
```

**Key design**: `ModuleCpu` is per-EU (isolated registers), but `ModuleMemory` and `ExportedModuleDictionary` are shared references from the parent `MbbsModule`.

### Constructor

Creates a new `CpuCore` and wires up:
- **Memory**: shared `IMemoryCore` from the module
- **ExternalFunctionDelegate**: routes `CALL FAR` to exported modules (see below)
- **Interrupt handlers**: `Int21h` (DOS file I/O), `Int3Eh` (MajorBBS-specific), `Int1Ah` (clock/date)
- **No I/O port handlers** (`ioPortHandlers: null`)

### ExternalFunctionDelegate

When the CPU executes a `CALL FAR` to a segment >= 0xFF00 (a virtual exported module segment), `CpuCore` fires the delegate instead of doing a normal far call:

```csharp
private ReadOnlySpan<byte> ExternalFunctionDelegate(ushort ordinal, ushort functionOrdinal)
{
    if (!ExportedModuleDictionary.TryGetValue(ordinal, out var exportedModule))
        throw new Exception($"Unknown or Unimplemented Imported Module: {ordinal:X4}");

    // Critical: re-associate this EU's registers with the exported module,
    // because nested EUs may have changed which registers it points to
    exportedModule.SetRegisters(ModuleCpuRegisters);

    return exportedModule.Invoke(functionOrdinal);
}
```

The `SetRegisters()` call is essential — because all EUs in a module share the same `ExportedModuleDictionary` (and thus the same `Majorbbs`/`Galgsbl` instances), a nested EU must re-bind its own registers before invoking. Without this, a nested call would read/write the parent EU's registers.

### Execute() — The Core Method

```csharp
public ICpuRegisters Execute(
    FarPtr entryPoint,              // CS:IP to start executing at
    ushort channelNumber,           // Which user session (or ushort.MaxValue for system calls)
    bool simulateCallFar = false,   // Push fake CS:IP return address for function pointer calls
    bool bypassState = false,       // Skip SetState() (for text variables, etc.)
    Queue<ushort> initialStackValues = null,  // Parameters to push before execution
    ushort initialStackPointer = CpuCore.STACK_BASE)  // SP start (shifted for nested calls)
```

**Step-by-step**:

1. **Reset CPU** — `ModuleCpu.Reset(initialStackPointer)` zeroes registers, sets SP/BP to the given stack pointer, pushes `uint.MaxValue` (0xFFFF:0xFFFF) as a halt sentinel onto the stack

2. **Set entry point** — `CS = entryPoint.Segment`, `IP = entryPoint.Offset`

3. **Push parameters** — dequeues `initialStackValues` onto the stack (if any)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mbbsemu/MBBSEmu](https://github.com/mbbsemu/MBBSEmu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
