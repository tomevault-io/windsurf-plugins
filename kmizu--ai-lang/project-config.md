---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ai-lang is a dependently-typed programming language implemented in Python. The language supports:
- Dependent types (types can depend on values)
- Basic data types (integers, booleans, strings)
- Collection types with size information (Vec n a)
- Pattern matching
- Type inference with bidirectional type checking

## Development Commands

```bash
# Install dependencies
poetry install

# Run tests
poetry run pytest

# Run specific test
poetry run pytest tests/test_lexer.py::test_function_name

# Type check with mypy
poetry run mypy src

# Format code
poetry run black src tests

# Lint code
poetry run ruff src tests

# Run the REPL
poetry run ai-lang
# Or directly:
PYTHONPATH=src python3 src/ai_lang/cli.py

# Run a file
poetry run ai-lang examples/minimal.ai
# Or directly:
PYTHONPATH=src python3 src/ai_lang/cli.py examples/minimal.ai
```

## Project Structure

```
ai-lang/
├── src/ai_lang/
│   ├── __init__.py       # Package initialization
│   ├── ast.py            # AST node definitions
│   ├── lexer.py          # Lexical analyzer
│   ├── parser.py         # Parser (to be implemented)
│   ├── typechecker.py    # Type checker (to be implemented)
│   ├── evaluator.py      # Interpreter (to be implemented)
│   └── cli.py            # Command-line interface
├── tests/                # Test suite
├── examples/             # Example programs
└── docs/
    └── syntax.md         # Language syntax specification
```

## Architecture

### Type System
- Based on dependent type theory with universe hierarchy
- Bidirectional type checking for better inference
- Support for implicit arguments
- Normalization by evaluation (NbE) for type equality

### Key Components
1. **Lexer**: Tokenizes source code, handles comments and layout
2. **Parser**: Builds AST using recursive descent or Lark grammar
3. **Type Checker**: Implements bidirectional type checking with:
   - Type synthesis (inferring types from terms)
   - Type checking (checking terms against types)
   - Unification for implicit arguments
4. **Evaluator**: Call-by-value interpreter with environments

### AST Design
- Immutable dataclasses with frozen=True
- Separate types for Type and Expr nodes
- Pattern matching support through Pattern hierarchy
- Source location tracking for error messages

## Implementation Status

### Completed Features
- ✅ Lexer with full token support
- ✅ Parser with comprehensive AST generation
- ✅ Core type system with dependent types
- ✅ Bidirectional type checker
- ✅ Evaluator/interpreter with pattern matching
- ✅ REPL with color support
- ✅ Basic data types (Nat, Bool, String)
- ✅ List literals with syntactic sugar
- ✅ Function definitions and calls
- ✅ Polymorphic types with implicit parameters
- ✅ Basic implicit type argument inference
- ✅ Explicit type application syntax `{Type}`
- ✅ Parameterized data types (e.g., `List A`)
- ✅ Module system with imports and exports
- ✅ **Constraint-based type inference for multiple implicit parameters**
- ✅ **Delayed inference for handling partial applications**
- ✅ **Implicit parameters accessible in function bodies**
- ✅ **Optimization passes: eta-reduction, dead code elimination, inlining**

### Known Issues and Limitations

1. **Implicit Type Inference** ✅ FULLY RESOLVED
   - ✅ Can now infer multiple implicit type parameters during type checking
   - ✅ Constraint-based solver successfully handles functions like `const : {A : Type} -> {B : Type} -> A -> B -> A`
   - ✅ Type checking correctly infers `const Z True` as `const {Nat} {Bool} Z True`
   - ✅ Direct evaluation now works correctly with inferred implicit arguments
   - ✅ The evaluator automatically elaborates terms during evaluation

2. **De Bruijn Indices** ✅ RESOLVED
   - ✅ Fixed issue with non-dependent function types in closures
   - ✅ Proper context extension for all function types
   - ✅ Fixed evaluator environment construction to match type checking context

3. **Type Conversion**
   - Data type names correctly convert to constructors in expression contexts
   - Added VPi quotation support for function types

## Side-Effect Handling ✅ IMPLEMENTED

A comprehensive IO monad system has been implemented for handling side effects:

### IO Monad System
- **Type-safe IO**: All side effects tracked in the type system via `IO` type
- **Monadic operations**: `pure` and `bind` for composing IO actions
- **Built-in IO primitives**:
  - `print`, `putStr`, `putStrLn` for output
  - `getLine` for input
  - Future support for file I/O operations
- **Dependent IO types**: IO types can depend on values (e.g., `readLines : (n : Nat) -> IO (Vec n String)`)
- **Effect tracking**: IO actions produce a list of effects for testing/debugging
- **Automatic execution**: Programs with `main : IO Unit` are automatically executed

### Example Usage:
```ai-lang
-- Hello World
main : IO Unit
main = putStrLn "Hello, World!"

-- Interactive program
main : IO Unit
main = bind {String} {Unit} getLine putStrLn
```

## All TODO Items Completed ✅

All planned features and improvements have been successfully implemented:

### Completed Features

1. **Implicit Type Inference** ✅ 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kmizu/ai-lang](https://github.com/kmizu/ai-lang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
