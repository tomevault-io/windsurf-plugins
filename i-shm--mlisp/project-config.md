---
trigger: always_on
description: Always use Context7 MCP when I need library/API documentation, code generation, setup or configuration steps without me having to explicitly ask.
---

# AGENTS.md

Always use Context7 MCP when I need library/API documentation, code generation, setup or configuration steps without me having to explicitly ask.

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

MLisp is a Lisp dialect interpreter implemented in OCaml. It features:
- S-expression syntax with prefix notation
- Lexical scoping and closures
- Hygienic macro system with quasiquote/unquote
- Module system for code organization
- REPL for interactive development

## Build and Development Commands

### Building
```bash
# From monorepo root
npm run build                  # Build all packages
npm run build:interpreter      # Build interpreter only
npm run build:vscode           # Build VSCode extension only

# Or from packages/interpreter/
cd packages/interpreter
opam install . --deps-only     # Install dependencies
dune build                      # Build the project
```

### Running
```bash
# Start the REPL (from monorepo root)
cd packages/interpreter && dune exec mlisp

# Run a MLisp file
cd packages/interpreter && dune exec mlisp -- <file.mlisp>

# Run a specific test file
cd packages/interpreter && dune exec mlisp -- test/<test-file>.mlisp
```

**Note**: When executing a file, the interpreter exits with code 1 if any errors occur. The REPL continues on errors.

### Testing
```bash
# From packages/interpreter/
cd packages/interpreter

# Using run_tests.sh (recommended - colored output, detailed errors)
./run_tests.sh                 # Run all tests
./run_tests.sh -v              # Verbose output
./run_tests.sh -s              # Stop on first failure
./run_tests.sh '0[1-5]*.mlisp' # Run specific pattern

# Using Makefile (from packages/interpreter/)
make test          # Run all tests
make test-verbose  # Run all tests with verbose output
make test-quick    # Run core tests (01-05)
make test-core     # Run core language tests (01-08)
make test-modules  # Run module system tests
make TEST=06_functions test-single  # Run single test
```

Tests are `.mlisp` files in the `test/` directory. The test runner (`run_tests.sh`) builds the project, runs each test file, and reports results with colored output.

### Development Workflow

When implementing new language features or modifying the interpreter:

1. **Read language reference** - Consult `README.md` for existing language syntax and semantics
2. **Write tests first** - Add or modify test files in `test/` covering the new functionality
3. **Run test suite** - Execute `./run_tests.sh` to verify all tests pass
4. **Update README.md** - If adding or modifying language features, update the Language Overview and related sections in `README.md`
5. **Check coverage** - Ensure the feature is covered by at least one test case

The test infrastructure detects:
- Non-zero exit codes (runtime errors)
- Error messages in output (`[error]`)
- Assertion failures (`Assertion failed`)
- Warnings (`[warning]`) - allowed for module tests, flagged for others

## Architecture

### Core Pipeline
The interpreter follows this data flow:
```
Input → Lexer → AST → Evaluator → Output
```

### Library Structure
The main `mlisp` library is composed of sub-libraries:
- **mlisp_utils** - Stream wrapper and string utilities
- **mlisp_repl** - REPL implementation with completion and hints
- **mlisp_stdlib** - Standard library loader

### Key Modules

| Directory | Purpose |
|-----------|---------|
| `lib/ast/` | Abstract Syntax Tree construction and parsing |
| `lib/lexer/` | Tokenization and lexical analysis |
| `lib/eval/` | Expression evaluation with quasiquote support |
| `lib/object/` | Core data types (lobject, environments, closures) |
| `lib/macro/` | Macro system with gensym for hygiene |
| `lib/primitives/` | Built-in functions (arithmetic, string, I/O) |
| `lib/stdlib/` | Standard library loader |
| `lib/repl/` | Read-Eval-Print Loop |
| `lib/module_loader/` | Module system implementation |
| `lib/error/` | Error handling and diagnostics (three exception types: `Syntax_error_exn`, `Parse_error_exn`, `Runtime_error_exn`) |
| `lib/print/` | Pretty-printing |
| `lib/utils/` | Utilities (stream handling, string utilities) |

### Entry Points
- `bin/mlisp.ml` - Main executable (REPL and file execution)
- `test/mlisp.ml` - Test runner

### REPL Features
The REPL (`lib/repl/repl.ml`) provides:
- **Tab completion** - Shows available bindings matching current input
- **Inline hints** - Shows type-aware suggestions as you type
- **History** - Persistent history in `.mlisp-repl-history`
- **Multi-line input** - Use `;;` to delimit expressions
- **Error context** - Shows source lines with error location highlighted

## Language Features

**Note**: `README.md` is the canonical reference for MLisp language syntax and semantics. Always consult README.md for the complete language specification before implementing interpreter changes.

### Quasiquote System
MLisp supports full quasiquote with nested quasiquotes:
- `` `expr `` - Quasiquote (literal content)
- `,expr` - Unquote (evaluate and insert)
- `,@expr` - Unquote-splicing (splice list)
- `` ` ``(expr) - Nested quasiquote (use `,,` for unquote at level 2)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [i-shm/MLisp](https://github.com/i-shm/MLisp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
