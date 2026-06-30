---
trigger: always_on
description: Provides debugging support for Luau scripts:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`luaz` is a zero-cost wrapper library for Luau written in Zig. It provides idiomatic Zig bindings for the Luau
scripting language, focusing specifically on Luau's unique features and performance characteristics.

The library entry point is `src/lib.zig`, which exports five public modules:
- Lua (`src/Lua.zig`): High-level idiomatic Zig API with automatic type conversions
- State (`src/State.zig`): Low-level Lua state wrapper providing direct access to Lua VM operations
- Compiler (`src/Compiler.zig`): Luau compiler interface for converting Lua source to bytecode
- Debug (`src/Debug.zig`): Debug functionality for breakpoints, stepping, and stack inspection
- GC (`src/GC.zig`): Garbage collector control for memory management tuning

## Build System

The build system is written in Zig and provides several targets:

### Building
- `zig build` - Build the library
- `zig build test` - Run unit tests
- `zig build luau-compile` - Run the Luau compiler binary
- `zig build luau-analyze` - Run the Luau analyze binary
- `zig build docs` - Generate and install documentation
- `zig build check-fmt` - Check code formatting
- `zig build luau-vm` - Build Luau VM library only
- `zig build luau-codegen` - Build Luau codegen library only
- `zig build guided-tour` - Run the guided tour example

### Key Libraries Built
- luau_vm: Core Luau virtual machine (from Luau's `VM/src`)
- luau_codegen: JIT code generation (from Luau's `CodeGen/src`) 
- luau_compiler: Luau compiler and AST (from Luau's `Compiler/src` and `Ast/src`)

The build system automatically discovers and compiles all `.cpp` and `.c` files in the respective Luau source
directories.

## Architecture

### High-Level API (`src/Lua.zig`)
The main `Lua` struct provides an idiomatic Zig interface with automatic type conversions:
- `init()` - Initialize Lua state with optional custom allocator
- `deinit()` - Clean up Lua state and free resources
- `openLibs()` - Open all standard Lua libraries (math, string, table, etc.)
- `enable_codegen()` - Enable Luau's JIT code generator for improved performance
- `globals()` - Access to the global environment table for setting/getting global variables
- `eval()` - Compile and execute Lua source code in one step
- `exec()` - Execute pre-compiled bytecode
- `createTable()` - Create new Lua tables with optional size hints
- `createMetaTable()` - Create metatable for a specific type with automatic method registration
- `createThread()` - Create new Lua thread/coroutine for concurrent execution
- `dumpStack()` - Debug utility to inspect the current Lua stack state
- `registerUserData()` - Register Zig structs as Lua userdata with automatic method binding
- `setAssertHandler()` - Set custom handler for Luau VM assertions
- `setCallbacks()` - Set VM callbacks for events like function calls and returns
- `sandbox()` - Enable sandbox mode for restricted execution environment
- `debug()` - Access debug functionality for breakpoints and tracing
- `gc()` - Access garbage collector control for memory management
- `top()` - Get current stack top position
- `status()` - Get coroutine/thread execution status
- `isYieldable()` - Check if current state can yield
- `reset()` / `isReset()` - Reset state to initial condition
- `getData()` / `setData()` - Manage user-defined data pointer
- `isThread()` - Check if this state is a thread/coroutine

### Type System Integration
The library provides seamless conversion between Zig and Lua types through its high-level API:
- Zig functions are automatically wrapped as callable Lua functions with argument type checking
- Optional types (`?T`) map to Lua nil values
- Tuples are converted to Lua tables with array semantics
- Reference system (`Ref`) allows holding Lua values across function calls
- Table wrapper (`Table`) provides safe access to Lua tables with automatic type conversion
- Function wrapper (`Function`) provides direct access to Lua functions with automatic type conversion
- Generic Value type for runtime Lua value handling when types are unknown at compile time
- Varargs iterator for handling variadic function arguments
- StrBuf for efficient string building and concatenation
- Result wrapper for function return values with multiple results support

### Table Operations
The library provides table operations through the `Table` type:
- Raw operations (`setRaw`/`getRaw`) bypass metamethods like `__index` and `__newindex` for direct table access
- Non-raw operations (`set`/`get`) invoke metamethods when present, providing full Lua semantics
- Closure setting with `Capture()` sets functions with upvalues for persistent state
- Function calling (`call`) retrieves and calls functions stored in tables with automatic argument and return type handling
- Function compilation (`compile`) compiles table functions to native code via JIT for better performance
- Table length (`len`) returns table length following Lua semantics, including metamethod support
- Table iteration (`iterator`) creates an iterator for traversing table entries
- Readonly control (`setReadonly`/`isReadonly`) manages table mutability
- Safe environment (`setSafeEnv`) marks table as safe execution environment

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mxpv/luaz](https://github.com/mxpv/luaz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
