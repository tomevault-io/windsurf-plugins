---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Onion is a statically-typed, object-oriented programming language that compiles to JVM bytecode. Originally written in Java, it has been rewritten in Scala (except the parser, which uses JavaCC).

**Configuration:**
- Scala version: 3.3.7
- Java version: 17
- SBT version: ~1.9+
- Main dependencies: ASM 9.8 (bytecode), JavaCC 5.0 (parser), ScalaTest 3.2.19 (testing)

## Build Commands

- **Compile project**: `sbt compile`
  - Automatically regenerates parser from `grammar/JJOnionParser.jj` if modified
- **Run tests**: `sbt test`
- **Run single test suite**: `sbt 'testOnly *HelloWorldSpec'`
- **Package JAR with dependencies**: `sbt assembly` (creates `onion.jar`)
- **Create distribution package**: `sbt dist` (creates ZIP with lib/, bin/, run/, onion.jar in target/)
- **Run Onion script**: `sbt 'runScript path/to/script.on [args]'`
- **Start REPL**: `sbt repl`
- **Clean build**: `sbt clean`
- **Scala console**: `sbt console`

### Compiler Options (for onionc/onion)

- `-classpath <path>` - Set classpath for compilation
- `-encoding <encoding>` - Set source file encoding
- `-d <dir>` - Set output directory for class files
- `-maxErrorReports <n>` - Limit number of errors reported
- `--dump-ast` - Print parsed AST to stderr
- `--dump-typed-ast` - Print typed AST summary to stderr
- `--warn <off|on|error>` - Set warning level
- `--Wno <codes>` - Suppress specific warnings (e.g., W0001,unused-parameter)

## High-Level Architecture

The Onion compiler is a **multi-phase pipeline compiler** following the classic compiler architecture:

### Compilation Pipeline

```
Source Files (.on)
    ↓
[1] Parsing (JavaCC) → Untyped AST
    ↓
[2] Rewriting → Normalized Untyped AST
    ↓
[3] Type Checking → Typed AST
    ↓
[4] Tail Call Optimization → Optimized Typed AST
    ↓
[5] Code Generation (ASM) → JVM Bytecode
    ↓
Class Loading & Execution
```

### Core Compiler Phases

All phases extend `Processor[A, B]` trait and can be composed using `andThen()`:

1. **Parsing** (`src/main/scala/onion/compiler/Parsing.scala`)
   - Uses JavaCC-generated parser from `grammar/JJOnionParser.jj` (36KB grammar file)
   - Parser class: `JJOnionParser` (auto-generated in `sourceManaged/`)
   - Converts source text → Untyped AST (`AST.scala`)
   - Uses `ASTBuilder.scala` to construct AST from parser tokens

2. **Rewriting** (`src/main/scala/onion/compiler/Rewriting.scala`)
   - AST transformation and normalization
   - Simplifies complex constructs
   - Output: Normalized untyped AST

3. **Type Checking** (`src/main/scala/onion/compiler/Typing.scala` + `typing/` package - largest component)
   - `Typing.scala` is a thin facade/orchestrator; the implementation lives in
     `src/main/scala/onion/compiler/typing/` (~60 focused classes)
   - Four sequential passes over all compilation units:
     1. `TypingHeaderPass` - registers classes/interfaces/records/enums/aliases
     2. `TypingOutlinePass` - members, inheritance, type parameters
     3. `TypingBodyPass` - types method/constructor bodies (delegates to
        `MethodCallTyping`, `OperatorTyping`, `ControlExpressionTyping`,
        `SelectExpressionTyping`, `TryExpressionTyping`, `ClosureTyping`, etc.)
     4. `TypingDuplicationPass` - duplicate/erasure-collision detection
   - State: global registries in `typing/session/TypingGlobalState`; per-unit
     mutable state in `typing/session/TypingUnitContext` (switched by `TypingSession.activate`)
   - Type inference, method resolution with overloading, access control
   - Key supporting files:
     - `ClassTable.scala` - Class symbol table
     - `LocalContext.scala` - Local variable environments
     - `SemanticErrorReporter.scala` - Error collection
   - Output: Typed AST (`TypedAST.scala`, 37KB)

4. **Tail Call Optimization** (`src/main/scala/onion/compiler/optimization/TailCallOptimization.scala`)
   - Detects tail-recursive methods (self-calls in return position)
   - Transforms tail recursion into loops to prevent stack overflow
   - Strategy:
     1. Copy parameters to loop variables at method entry
     2. Rewrite all parameter references to use loop variables
     3. Wrap method body in `while(true)` loop
     4. Replace tail calls with variable updates + continue
   - Prevents StackOverflowError for deep recursion (e.g., 10000+ calls)
   - Output: Optimized Typed AST

5. **Code Generation** (`src/main/scala/onion/compiler/AsmCodeGeneration.scala`, 42KB)
   - **ASM-based bytecode generation** (current implementation)
   - Visitor pattern: `AsmCodeGenerationVisitor.scala`
   - Bytecode utilities:
     - `bytecode/MethodEmitter.scala` - JVM method generation
     - `bytecode/LocalVarContext.scala` - Local variable tracking
     - `bytecode/AsmUtil.scala` - ASM helper functions
   - Output: `CompiledClass` objects (in-memory or file)

### Key Architectural Components

**Orchestration:**
- `OnionCompiler.scala` - Main compiler orchestrator, composes all phases
- Returns `CompilationOutcome` (Success with classes or Failure with errors)

**Entry Points:**
- `onion.tools.CompilerFrontend` - CLI for `onionc` command (compile to .class files)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [onion-lang/onion](https://github.com/onion-lang/onion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
