---
trigger: always_on
description: handleSetError :: Error -> String
---

# Acton Compiler - Development Guide

The Acton compiler (`acton`) is written in Haskell and compiled Acton source code into C code that can be compiled and linked with the runtime system.

## Quick Reference

### Build & Test
```bash
# Build just the compiler (fastest iteration)
make dist/bin/acton

# Run compiler tests
make test-compiler

# Run specific test suite
cd compiler && stack test --test-arguments "--filter pattern"

# Build with profiling
cd compiler && stack build --profile
```

### Key Directories
```
compiler/
├── lib/                    # Main compiler library
│   ├── src/Acton/         # Core compiler modules
│   └── test/              # Compiler unit tests
├── acton/                 # Compiler executable
│   ├── Main.hs           # Entry point
│   └── test/             # Snapshot tests
└── lsp-server/           # Language server
```

## Compilation Pipeline

The compiler transforms Acton code through multiple stages:

```
Source Code (.act)
    ↓ Parser
AST (Abstract Syntax Tree)
    ↓ Type Checker
Typed AST
    ↓ Normalizer
Normalized AST
    ↓ Deactorizer
Deactorized AST
    ↓ CPS Transform
CPS Form
    ↓ Lambda Lifter
Lifted Form
    ↓ Boxing
Boxed Form
    ↓ Code Generator
C Code (.c, .h)
```

## Key Modules

### Core Pipeline (`compiler/lib/src/Acton/`)

#### Parser.hs
- Megaparsec-based parser
- Produces AST from source code
- Handles Python-like syntax with Acton extensions
- Entry point: `parseModule`

#### Types.hs & TypeEnv.hs
- Type system definitions
- Type inference engine
- Constraint solving
- Key types: `Type`, `Scheme`, `TyEnv`

#### Normalizer.hs
- Simplifies AST for easier processing
- Desugars complex expressions
- Handles pattern matching expansion

#### Deactorizer.hs
- Transforms actor methods into regular functions
- Converts actor state into explicit parameters
- Handles message passing semantics

#### CPS.hs
- Continuation-Passing Style transformation
- Makes control flow explicit
- Prepares for async operations

#### LambdaLifter.hs
- Lifts nested functions to top level
- Closure conversion
- Prepares for C code generation

#### Boxing.hs
- Converts between boxed/unboxed representations
- Handles primitive type optimizations

#### CodeGen.hs
- Generates C code from final AST
- Produces `.c` and `.h` files
- Integrates with runtime system

### Supporting Modules

#### Env.hs
- Environment management
- Module system handling
- Import resolution

#### Diagnostics.hs
- Error message generation
- Source location tracking
- Pretty printing errors

#### QuickType.hs
- Fast type checking for IDE support
- Incremental type inference

## Working with the Compiler

### Adding a New Language Feature

1. **Update Parser** (`Parser.hs`)
   - Add new syntax rules
   - Update AST types in `Syntax.hs`

2. **Update Type System** (`Types.hs`)
   - Add type rules for new construct
   - Update type inference

3. **Update Normalizer** (`Normalizer.hs`)
   - Add normalization rules
   - Ensure simplified form

4. **Update Code Generation** (`CodeGen.hs`)
   - Generate appropriate C code
   - Link with runtime if needed

5. **Add Tests**
   - Parser tests in `lib/test/`
   - Type error tests in `acton/test/typeerrors/`
   - Snapshot tests for code generation

### Debugging the Compiler

```haskell
-- Add debug prints in any module
import Debug.Trace

-- In your code
traceShow ("Debug info", someValue) $ restOfExpression

-- Or use the built-in pretty printer
import Pretty
traceShow (renderDoc $ pp someAst) $ ...
```

### Testing Documentation Generation

Documentation generation tests are located in `test/test_doc_printing/`. This directory contains various test files demonstrating different documentation scenarios.

To test documentation generation:
```bash
acton doc src/basics.act

# Whole project
acton doc
```

### Error Messages

Error messages are crucial for user experience. When adding new errors:

1. Create descriptive error in `Diagnostics.hs`
2. Add snapshot test in `acton/test/typeerrors/`
3. Include:
   - Clear description of the problem
   - Source location
   - Suggested fix if possible

Example:
```haskell
throwError $ TypeError loc $ 
  "Cannot unify types" <+> pp ty1 <+> "and" <+> pp ty2
```

## Type System Details

### Type Inference
- Hindley-Milner with extensions
- Row polymorphism for records
- Subtyping for actors
- Protocol constraints

### Special Types
- `World` - Represents side effects
- `Msg` - Actor message types
- `Cap` - Capability types
- `Ref` - Reference types

## Testing Strategy

### Unit Tests (`lib/test/`)
- Test individual compiler phases
- Use sydtest framework
- Fast, focused tests

### Snapshot Tests (`acton/test/`)
- Compare compiler output against expected
- Syntax errors: `syntaxerrors/`
- Type errors: `typeerrors/`
- Update with: `--accept` flag

### Integration Tests
- Full compilation tests in `test/`
- Ensure generated C code works

## Common Tasks

### Adding a Builtin Function
1. Add signature in `Builtin.hs`
2. Add type in builtin environment
3. Generate C call in `CodeGen.hs`

### Improving Error Messages
1. Identify error location in type checker
2. Add case in `Diagnostics.hs`
3. Create snapshot test
4. Iterate on message clarity

### Optimizing Compilation
1. Profile with `stack build --profile`
2. Run with `+RTS -p`
3. Check `.prof` file

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [actonlang/acton](https://github.com/actonlang/acton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
