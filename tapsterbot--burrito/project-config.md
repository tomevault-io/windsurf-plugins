---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains **Burrito** (v0.3.0), a unified Nim wrapper for multiple scripting engines including QuickJS JavaScript engine (version 2025-04-26) and MicroPython. QuickJS is a small and embeddable JavaScript engine that supports ES2023 specification, while MicroPython provides a lean Python implementation using the official embedding API.

**🎯 THE KILLER FEATURE:** Burrito's primary value proposition is the ability to embed complete scripting language REPLs with syntax highlighting, command history, and custom Nim function exposure into your applications with just a few lines of code.

The Burrito wrappers are located in `src/burrito/` and provide comprehensive, idiomatic Nim interfaces with automatic memory management and extensive API coverage for both engines.

## Quick Start

### Installation
```bash
# Clone and setup Burrito
git clone https://github.com/tapsterbot/burrito.git
cd burrito

# Download and build QuickJS
nimble get_quickjs
nimble build_quickjs

# Download and build MicroPython
nimble get_micropython
nimble build_micropython

# Test installation
nimble example
```

### Basic Usage Example
```nim
import burrito/qjs  # or: import burrito/mpy for MicroPython

var js = newQuickJS()
echo js.eval("3 + 4")                    # Output: 7
echo js.eval("'Hello ' + 'World!'")      # Output: Hello World!

proc greet(ctx: ptr JSContext, name: JSValue): JSValue =
  let nameStr = toNimString(ctx, name)
  result = nimStringToJS(ctx, "Hello from Nim, " & nameStr & "!")

js.registerFunction("greet", greet)
echo js.eval("greet('Burrito')")         # Output: Hello from Nim, Burrito!
js.close()
```

### MicroPython Usage Example
```nim
import burrito/mpy

var py = newMicroPython()
echo py.eval("print(3 + 4)")             # Output: 7
echo py.eval("print('Hello ' + 'World!')")  # Output: Hello World!
py.close()
```

### Embedded REPL Example
```nim
import burrito/qjs

var js = newQuickJS(configWithBothLibs())

proc greet(ctx: ptr JSContext, name: JSValue): JSValue =
  let nameStr = toNimString(ctx, name)
  result = nimStringToJS(ctx, "Hello from Nim, " & nameStr & "!")

js.registerFunction("greet", greet)

# Option 1: Load REPL from file
let replCode = readFile("quickjs/repl.js")
discard js.evalModule(replCode, "<repl>")
js.runPendingJobs()
js.processStdLoop()  # Interactive REPL runs here!
js.close()

# Option 2: Use pre-compiled bytecode (faster, no file dependency)
# First run: nimble compile_repl_bytecode
import ../build/src/repl_bytecode
discard js.evalBytecodeModule(qjsc_replBytecode)
js.runPendingJobs()
js.processStdLoop()
js.close()
```

## Nimble Commands

### Development
```bash
nimble example          # Run basic QuickJS example
nimble examples         # Run all QuickJS examples
nimble repl_js          # Start QuickJS REPL with custom Nim functions
nimble test_report      # Run test suite with summary
nimble compile_repl_bytecode  # Compile repl.js to bytecode
```

### QuickJS Management
```bash
nimble get_quickjs     # Download QuickJS source
nimble build_quickjs   # Build QuickJS library
nimble delete_quickjs  # Remove QuickJS source
nimble clean_all       # Clean all build artifacts
```

### MicroPython Management
```bash
nimble get_micropython     # Download MicroPython source
nimble build_micropython   # Build MicroPython embedding library
nimble delete_micropython  # Remove MicroPython source
nimble example_mpy         # Run basic MicroPython example
nimble repl_mpy            # Interactive REPL with readline support
nimble empy                # Alias for example_mpy

# Multi-engine examples
nimble dual_engines        # Run JavaScript + Python together
```

### Documentation
```bash
nimble docs           # Generate API documentation
nimble serve_docs     # Serve docs locally at http://localhost:8000
```

## Project Structure

### Core Files
- `src/burrito.nim` - Main module that re-exports both QuickJS and MicroPython modules
- `src/burrito/qjs.nim` - QuickJS wrapper with comprehensive bindings
- `src/burrito/mpy.nim` - MicroPython wrapper using embedding API
- `burrito.nimble` - Package configuration with all development tasks
- `nim.cfg` - Build configuration (outputs to build/bin/)
- `docs/index.html` - Modern landing page with sunset gradient theme
- `docs/burrito.html` - Generated API documentation
- `docs/mpy/MICROPYTHON.md` - Detailed MicroPython integration documentation
- `docs/mpy/micropython_rom_levels_analysis.md` - ROM level configuration analysis
- `docs/qjs.html` - QuickJS module API documentation
- `docs/mpy.html` - MicroPython module API documentation
- `docs/mpy/MICROPYTHON.md` - Detailed MicroPython integration notes
- `docs/mpy/micropython_rom_levels_analysis.md` - ROM level configuration analysis

### QuickJS Examples (in `examples/qjs/`)
- `basic_example.nim` - Simple JavaScript evaluation
- `call_nim_from_js.nim` - Exposing Nim functions to JavaScript
- `bytecode_basic.nim` - Basic bytecode compilation and execution
- `bytecode_comprehensive.nim` - Complete bytecode feature showcase
- `repl_qjs.nim` - Standalone REPL implementation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tapsterbot/burrito](https://github.com/tapsterbot/burrito) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
