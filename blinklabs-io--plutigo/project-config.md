---
trigger: always_on
description: This document provides comprehensive guidance for AI agents working on the plutigo codebase.
---

# Agent Guide for Plutigo

This document provides comprehensive guidance for AI agents working on the plutigo codebase.

## Quick Start

```bash
# Verify environment
go version          # Must be 1.25+ (1.26+ recommended)
make test           # Run all tests
golangci-lint run   # Lint check
```

## Project Overview

**plutigo** is a pure Go implementation of Untyped Plutus Core (UPLC), the smart contract VM for Cardano blockchain.

### Key Concepts

| Term | Definition |
| ------ | ---------- |
| UPLC | Untyped Plutus Core - the low-level language scripts compile to |
| CEK Machine | The evaluation engine (Control-Environment-Continuation) |
| Plutus Version | V1, V2, V3, V4 - different eras with different features |
| ExBudget | Execution budget (CPU + memory units) |
| PlutusData | The data format for script inputs/outputs |
| Builtin | Built-in functions like `addInteger`, `sha2_256` |

### Architecture

```text
Input UPLC → Parse (syn/) → De Bruijn → CEK Machine (cek/) → Result
                                              ↓
                                    Cost Model (budget tracking)
```

---

## Package Guide

### `cek/` - Evaluation Engine (Primary)

The CEK machine that executes UPLC programs.

| File | Purpose |
| ------ | ------- |
| `machine.go` | Core CEK machine loop, state management |
| `builtins.go` | 104 builtin implementations |
| `cost_model.go` | Budget tracking, cost model types |
| `cost_model_builtins.go` | Per-builtin cost functions |
| `env.go` | Environment (variable bindings) |
| `value.go` | Runtime value types |

**Standard evaluation flow:**
```go
program, _ := syn.Parse(input)
dbProgram, _ := syn.NameToDeBruijn(program)
machine := cek.NewMachine[syn.DeBruijn](dbProgram.Version, 0, nil)
result, _ := machine.Run(dbProgram.Term)
```

### `syn/` - Syntax and Parsing

| File | Purpose |
| ------ | ------- |
| `parser.go` | UPLC text parser |
| `ast.go` | Abstract syntax tree types |
| `debruijn.go` | Name to De Bruijn index conversion |
| `flat/` | FLAT binary serialization |

### `builtin/` - Builtin Definitions

| File | Purpose |
| ------ | ------- |
| `builtins.go` | DefaultFunction enum (104 builtins) |
| `availability.go` | Which builtins available in which version |
| `arity.go` | Argument counts |
| `force_count.go` | Type instantiation counts |

### `data/` - PlutusData Codec

CBOR encoding/decoding for Plutus data types.

### `lang/` - Language Constants

| File | Purpose |
| ------ | ------- |
| `version.go` | Language version constants |
| `v1.go`, `v2.go`, `v3.go` | Cost model parameter names |
| `v4.go` | **MISSING** - needs creation (Plan 11) |

---

## Development Workflow

### Before Starting Any Task

1. **Pull latest:** `git pull origin main`
2. **Understand the task:** Review related code and tests
3. **Understand acceptance criteria:** Know what "done" means

### While Working

1. **Write tests first** when feasible
2. **Run tests incrementally:** `make test-match TEST=YourTest`
3. **Keep changes focused:** One logical change per commit

### Before Completing

```bash
# Required checks
make test                    # Must pass
golangci-lint run ./...      # Must have 0 issues
nilaway ./...                # Nil safety check

# Recommended
make bench                   # If touching hot paths
```

### Commit Convention

Format: `type: description`

| Type | Use For |
| ------ | ------- |
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Code restructuring (no behavior change) |
| `test` | Test additions/changes |
| `docs` | Documentation |
| `chore` | Build, deps, CI |

**Always sign commits:** `git commit -s -m "type: message"`

---

## Code Patterns

### Table-Driven Tests

```go
func TestFoo(t *testing.T) {
    tests := []struct {
        name     string
        input    string
        expected string
    }{
        {"case1", "input1", "expected1"},
        {"case2", "input2", "expected2"},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Foo(tt.input)
            if result != tt.expected {
                t.Errorf("got %v, want %v", result, tt.expected)
            }
        })
    }
}
```

### Error Handling

The codebase uses typed errors for machine failures:
```go
return nil, &BudgetError{Code: ErrCodeBudgetExhausted, Requested: cost, Available: remaining, Message: "out of budget"}
return nil, &BuiltinError{Code: ErrCodeUnimplemented, Builtin: "caseList", Message: "unimplemented"}
return nil, &InternalError{Code: ErrCodeInternalError, Message: "compute returned nil state"}
```

### Builtin Implementation Pattern

```go
func (m *Machine[T]) builtinAddInteger(args []Value) (Value, error) {
    a := args[0].(*Int)
    b := args[1].(*Int)
    return &Int{Value: new(big.Int).Add(a.Value, b.Value)}, nil
}
```

---

## Common Tasks

### Adding a New Builtin

1. Add constant to `builtin/builtins.go`
2. Update `MaxDefaultFunction`
3. Add to `builtinNames` map
4. Add availability in `builtin/availability.go`
5. Add arity in `builtin/arity.go`
6. Add force count in `builtin/force_count.go`
7. Implement in `cek/builtins.go`
8. Add cost model in `cek/cost_model_builtins.go`
9. Add conformance tests to `tests/`

### Adding a Cost Model


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blinklabs-io/plutigo](https://github.com/blinklabs-io/plutigo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
