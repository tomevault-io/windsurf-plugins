---
trigger: always_on
description: This file defines mandatory development rules for `unidecompiler`.
---

# Agent Development Rules

This file defines mandatory development rules for `unidecompiler`.

The project is built around one hard architectural rule: VM frontends are thin
submitters, and the core owns recovery.

## Architecture Contract

The decompiler is split into three layers:

1. VM frontends parse bytecode formats and submit neutral thin IR.
2. Core lifts thin VM steps, effects, hints, regions, CFG-like control flow, and
   recoverable structures.
3. Backends render the recovered generic IR into pseudocode.

The frontend pipeline is:

1. Decode the VM bytecode with the frontend's format decoder.
2. Convert each decoded instruction into a `VMBytecodeStep`.
3. Attach neutral operands, opcode classes, hints, and effect-table results.
4. Submit the complete step stream through `lift_vm_step_function`.
5. Let core produce full, partial, or unsupported generic IR.
6. Treat any unsupported result in an intended coverage path as a defect to
   eliminate, not an acceptable development endpoint.

The full recovery pipeline is:

```txt
VM bytecode -> thin IR -> generic IR -> SSA/analysis -> AST -> pseudocode
```

Useful source metadata should flow through the pipeline when available,
including source filenames, bytecode versions, constants, debug tables,
instruction offsets, line info, local variable info, upvalue or member names,
and frontend diagnostics. This metadata is provenance and analysis context; it
must not become a reason for core to depend on frontend-private decoded models.
Metadata is pass-through context only. It must not express program logic,
control flow, recovery decisions, or source-language semantics.

Here, `unsupported` means the core could not safely recover the current stack
shape, control-flow shape, or IR combination, so it must emit an explicit
fallback instead of guessing.
This fallback exists only as a safety valve. During active development, any
`unsupported` produced for a supported or intended-to-be-supported shape must be
treated as a bug and removed before the change is considered done.
An `unsupported` result must include enough bytecode context to analyze the
failure: the relevant instruction window, raw opcode text when available,
decoded operands, branch targets or region hints, and the reason recovery
stopped.
If even that fallback cannot be expressed safely, the core should raise an
explicit error rather than emitting misleading pseudocode.

Python `.pyc`, JVM `.class`, Lua chunks, .NET CLI assemblies, and WebAssembly
modules follow this model. JVM class reading uses the `jawa` library and must
not shell out to `javap`. .NET/C# assembly reading uses the `dnfile` library and
must not shell out to disassembly tools. WASM reading uses library validation
and instruction decoding (`wasmtime` + `wasm`) before submitting thin operators.

The stress corpus is organized as `opcode_projects/source/<project>` and
`opcode_projects/generate/<project>`. Generated and source stress projects are
local working data; they are scanned by path rather than imported as a Python
test package.

## Simulation Architecture Contract

Simulation is a separate, optional consumer of recovered generic IR. Its
architectural boundary is as strict as the frontend/core boundary: preserving
this decoupling is mandatory.

The simulation dependency direction is:

```txt
frontend -> core generic IR <- simulator <- CLI / GUI / other application hosts
```

`core` must not import, depend on, or know about the simulator. The simulator
may depend on public generic IR, but it must not execute frontend bytecode,
decoded frontend models, VM opcodes, opcode effect tables, or thin IR effects.
It owns generic-IR frames, calls, control flow, limits, exceptions,
cancellation, and execution tracing. It must not branch on a frontend ID or
contain language-specific execution behavior.

A frontend may optionally expose a `simulation_adapter`. Unsupported simulation
is valid. A frontend that opts in may only:

- enumerate presentation-safe, data-only target queries;
- resolve a frontend-owned query to a `FunctionIR` belonging to the current
  lifted `ModuleIR`, including frontend-specific ambiguity handling; and
- supply narrow, data-only runtime facts when generic IR cannot express them.

Simulation adapters must not execute or interpret functions or instructions,
maintain frames or stacks, recover control flow, inspect simulator internals, or
return executable callbacks. The simulator validates that a resolved function
belongs to the current lifted module and retains sole ownership of execution.
CLI and GUI treat frontend target queries as opaque data; they must not infer
language-specific function names, overload resolution, class/member lookup, or
dynamic call targets.

Unresolved named calls may be delegated only through the data-only
`ExternalEnvironment` protocol. An environment receives an
`ExternalCallRequest` and returns an `ExternalCallResult`; it must never
receive generic IR, frames, adapters, or execution control. Its inputs and
outputs must be validated generic runtime values. A runtime file such as
`runtime.py` is explicitly selected, trusted host code: it is not sandboxed and
its loading belongs in a host-support package, never in core, the simulator, or
a frontend.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Wker666/unidecompiler](https://github.com/Wker666/unidecompiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
