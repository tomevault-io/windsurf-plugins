---
trigger: always_on
description: **Note:** This file contains fortfront-specific guidance. General Fortran/Git/fpm/GitHub rules are in the user's global CLAUDE.md (`~/.claude/CLAUDE.md`) and always apply.
---

# CLAUDE.md

**Note:** This file contains fortfront-specific guidance. General Fortran/Git/fpm/GitHub rules are in the user's global CLAUDE.md (`~/.claude/CLAUDE.md`) and always apply.

## Instruction Precedence (fortfront)

- The user's global CLAUDE.md at `~/.claude/CLAUDE.md` defines general rules for Fortran, Git, fpm, and GitHub.
- This fortfront `CLAUDE.md` refines those rules for this repository and is the canonical policy document here.
- When this file disagrees with other project documentation, scripts, or comments in this repo, this file wins.
- `AGENTS.md` in this repository must be a symlink to this file so there is exactly one source of truth.

## What is fortfront?

Fortfront is a **Fortran frontend library** that parses and analyzes **both standard Fortran and Lazy Fortran**. It provides a complete AST, semantic analysis, and type inference infrastructure for building tools like:

- **Linters and formatters** (fluff)
- **Compilers** (LLVM HLIR emission)
- **Static analyzers**
- **Language servers**
- **Code transformation tools**

**On its own**, fortfront can also **standardize Lazy Fortran** to standard Fortran via CLI and API.

**Lazy Fortran transformation example:**
```fortran
! Input: script.lf (lazy fortran - minimal syntax)
function add(a, b)
    add = a + b
end function
x = add(5, 3)

! Output: standard Fortran (inferred types, intents, structure)
program main
    implicit none
    integer :: x
contains
    integer function add(a, b)
        integer, intent(in) :: a, b
        add = a + b
    end function
end program
```

**Standard Fortran round-trip:** `.f90` → parse → AST → emit → `.f90` (validates correctness)

## Examples & Tests Organization

### CRITICAL: Zero Duplication Policy - MANDATORY ENFORCEMENT
**ONE canonical example, many references. NO DUPLICATION EVER.**

This is not a suggestion - it is an **absolute requirement**. Violations block PRs and must be fixed immediately.

### EXCEPTION: Unit Tests vs Integration Tests - BOTH ARE REQUIRED

**Unit tests with inline code are ENCOURAGED**. The zero-duplication policy applies to **END-TO-END tests ONLY**.

**Test Hierarchy (ALL levels required)**:
1. **Unit Tests** - Test individual functions/modules in isolation
   - ✅ **Inline code ALLOWED and ENCOURAGED** for small, focused test inputs
   - ✅ Test one function, one module, one specific code path
   - ✅ Fast, targeted, specific to the unit under test
   - Example: Testing a parser function with a 2-line input string

2. **Integration Tests** - Test interactions between components
   - ⚠️ **Use judgment**: Small inline snippets OK, large programs use examples/
   - Test multiple components working together
   - Example: Parser + semantic analyzer working on a small construct

3. **End-to-End Tests** - Test complete pipeline with realistic programs
   - ❌ **MUST use examples/**: These test complete transformation workflows
   - ❌ **NO inline code**: Full programs belong in examples/
   - Test: lexer → parser → semantic → codegen → compile → run
   - Example: Full lazy Fortran program → standardized Fortran output

**Decision Tree**:
```
Is this testing a SINGLE UNIT (function/module)?
├─ YES → Unit test with inline code is PERFECT ✅
└─ NO → Is it testing a complete program transformation?
    ├─ YES → MUST use examples/ (end-to-end) ❌ inline
    └─ NO → Integration test: use judgment (prefer examples/ for >10 lines)
```

### Directory Structure (STRICT)
```
examples/
├── f90/          # Standard Fortran examples (canonical source of truth)
│   ├── *.f90     # Round-trip validation examples
│   ├── issue_NNNN_description.f90  # Issue-specific standard Fortran
│   └── feature_name.f90            # Feature demonstrations
│
└── lf/           # Lazy Fortran examples (canonical source of truth)
    ├── *.lf      # Type inference and standardization examples
    ├── issue_NNNN_description.lf   # Issue-specific lazy Fortran
    └── feature_name.lf             # Feature demonstrations

test/
├── api/          # API tests (MUST use read_example)
├── ast/          # AST tests (MUST use read_example)
├── analysis/     # Analysis tests (MUST use read_example)
├── codegen/      # Codegen tests (MUST use read_example)
├── integration/  # Integration tests (MUST use read_example)
│   ├── issue_tests/      # Issue-specific test logic
│   ├── core_features/    # Core feature test logic
│   └── array_tests/      # Array feature test logic
└── *.f90         # All test files REFERENCE examples/, NEVER inline code
```

### Mandatory Rules - ZERO TOLERANCE

#### 1. **Examples are canonical sources (ENFORCED)**
   - `examples/` contains THE ONLY definitive example code
   - Examples demonstrate features, edge cases, and issue resolutions
   - Named descriptively: `generic_functions.lf`, `array_syntax.lf`, NOT `test_*.lf`
   - Issue demonstrations: `issue_NNNN_description.lf` (keep issue number for traceability)
   - Examples are documentation AND test inputs - dual purpose by design

#### 2. **End-to-End Tests MUST reference examples (ZERO INLINE CODE for full programs)**
   - **ABSOLUTELY FORBIDDEN in end-to-end tests**: Full program inline code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lazy-fortran/fortfront](https://github.com/lazy-fortran/fortfront) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
