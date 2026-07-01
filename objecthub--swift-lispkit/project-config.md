---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LispKit is a framework for building Lisp-based extension and scripting languages for macOS and iOS applications. It implements a core language based on the R7RS (small) Scheme standard, fully written in Swift 6. The framework consists of a compiler, virtual machine, garbage collector, and extensive library system.

## Build Commands

### Using Swift Package Manager

**Build debug binary:**
```bash
swift build -Xswiftc "-D" -Xswiftc "SPM"
```

**Build release binary:**
```bash
swift build -c release -Xswiftc "-D" -Xswiftc "SPM"
```

**Run REPL (debug):**
```bash
.build/debug/LispKitRepl -r Sources/LispKit/Resources -d LispKit
```

**Run REPL (release):**
```bash
.build/release/LispKitRepl -r Sources/LispKit/Resources -d LispKit
```

**Execute a Scheme program:**
```bash
.build/debug/LispKitRepl -r Sources/LispKit/Resources -d LispKit path/to/program.scm
```

### Using Makefile

**Build and run debug REPL:**
```bash
make run
```

**Build and run release REPL:**
```bash
make repl
```

**Execute a specific program:**
```bash
make run program=path/to/program.scm
```

**Run tests:**
```bash
make test
# or
swift test -Xswiftc "-D" -Xswiftc "SPM"
```

**Clean build artifacts:**
```bash
make clean
```

### Using Xcode

Open `LispKit.xcodeproj` and build using the appropriate scheme:
- **LispKit**: Core framework for macOS
- **LispKit iOS**: Core framework for iOS
- **LispKitTools**: REPL framework for macOS
- **LispKitRepl**: Command-line tool for macOS
- **LispKitRepl iOS**: iOS application with chat-style UI

## Architecture

### Core Components

**Context** (`Sources/LispKit/Runtime/Context.swift`)
- Central hub connecting all interpreter components
- Manages shared resources: heap, symbols, libraries, file handler
- Owns the evaluator, virtual machine, and global environment
- Use `LispKitContext` for standard setup or extend `Context` for customizations

**Compiler** (`Sources/LispKit/Compiler/Compiler.swift`)
- Translates Scheme expressions into bytecode
- Two-phase compilation with optional optimization
- Manages environments, captures, and constant pools
- Produces `Code` objects containing bytecode and metadata

**VirtualMachine** (`Sources/LispKit/Runtime/VirtualMachine.swift`)
- Stack-based bytecode interpreter
- Handles tail calls, continuations (`call/cc`), and `dynamic-wind`
- Frame-based stack layout with procedure, arguments, locals, and stack values
- Includes garbage collector triggering logic

**LibraryManager** (`Sources/LispKit/Runtime/LibraryManager.swift`)
- Manages loading and registration of both native (Swift) and Scheme libraries
- Lazy loading: libraries are loaded on first `lookup`
- Libraries are stored in `Sources/LispKit/Resources/Libraries/` directory

**Expr** (`Sources/LispKit/Data/`)
- Core data representation for all LispKit values
- Implemented as Swift enum with cases for all Scheme types
- Located in `Sources/LispKit/Data/` - see `Exprs.swift`, `Cell.swift`, `Symbol.swift`, etc.

### Directory Structure

**Documentation/**
- Contains documentation in Markdown format for all LispKit libraries
- The libraries are typically implemented in Sources/LispKit/Primitives

**Tests/**
- Contains the code for all unit tests

**Tests/LispKitTests/**
- Contains the code for all unit tests of the LispKit core framework
- All code is written in Swift

**Tests/LispKitTests/Code/**
- Contains parts of the unit tests that are written in Scheme
- This code is testing built-in libraries

**Sources/**
- Contains all source code for LispKit

**Sources/LispKit**
- Contains all source code for the core framework

**Sources/LispKit iOS/**
- Contains an iOS-specific configuration files

**Sources/LispKitTools/**
- Contains a macOS-specific framework for building read-eval-print (REPL) loops with a terminal-based command-line interface
- Command-line argument parsing for the REPL

**Sources/LispKitRepl/**
- Contains a macOS-specific implementation of a read-eval-print (REPL) command-line interface

**Sources/LispKitRepl iOS/**
- Contains an iOS-specific implementation of a read-eval-print (REPL) interface for iPhones
- iOS application with chat-style UI
- This is typically used in a simulator

**Sources/LispKit/Base/**
- Utilities: object pools, locks, timers, string builders
- Foundation for managing memory and threading

**Sources/LispKit/Compiler/**
- Scanner, parser, compiler, error types
- Environment (`Env`) and binding management

**Sources/LispKit/Data/**
- Core data structures: `Expr`, `Symbol`, `Cell`, `HashTable`, `Port`, `Procedure`, etc.
- Type system and equality/hashing implementations

**Sources/LispKit/Runtime/**
- Virtual machine, context, library manager, evaluator
- Garbage collection (`Heap.swift`)
- Code execution infrastructure

**Sources/LispKit/IO/**
- Port system: text and binary input/output
- Abstractions for file, string, and custom I/O sources

**Sources/LispKit/Graphics/**
- Drawing, shapes, colors, transformations
- Platform-specific implementations (separate iOS variants)

**Sources/LispKit/Primitives/**
- Native library implementations (`*Library.swift` files)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [objecthub/swift-lispkit](https://github.com/objecthub/swift-lispkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
