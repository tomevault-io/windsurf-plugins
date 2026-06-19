---
trigger: always_on
description: You are an expert C++ and TypeScript developer working on `ts-aot`, an Ahead-of-Time compiler for TypeScript.
---

# TypeScript AOT Compiler (ts-aot)

You are an expert C++ and TypeScript developer working on `ts-aot`, an Ahead-of-Time compiler for TypeScript.

## Project Overview

This is an LLVM-based ahead-of-time compiler that compiles TypeScript directly to native executables. The compiler generates optimized machine code while maintaining JavaScript semantics and Node.js API compatibility.

**Current Status:** Conformance-driven development

## Essential Documentation

**Read these at the start of every session:**
- @.github/context/active_state.md - Current phase, recent accomplishments, active tasks
- @.github/instructions/conformance-workflow.instructions.md - **Feature implementation workflow**

**Conformance Matrices (feature tracking):**
- @docs/conformance/typescript-features.md - TypeScript features (174 total, 41% implemented)
- @docs/conformance/ecmascript-features.md - ECMAScript features (223 total, 36% implemented)
- @docs/conformance/nodejs-features.md - Node.js APIs (610 total, 20% implemented)

**Reference documentation (consult as needed):**
- @.github/DEVELOPMENT.md - Detailed development guidelines
- @.github/context/architecture_decisions.md - Key architectural choices
- @.github/context/known_issues.md - Current limitations and technical debt
- @.github/instructions/quick-reference.md - Quick lookup for common patterns
- @.github/instructions/code-snippets.md - Copy-paste ready code templates
- @.github/instructions/runtime-extensions.instructions.md - How to extend the runtime
- @.github/instructions/adding-nodejs-api.instructions.md - How to add Node.js APIs

## Skills Available

This project includes Claude Code skills for automated tasks:

### Auto-Debug Skill (`/auto-debug`)
**Trigger terms:** crash, access violation, debug, analyze crash, CDB, debugger
**Location:** `.claude/skills/auto-debug/`

Automatically analyzes crashes using CDB debugger. Extracts stack traces, exception info, and crash locations.

**⚠️ MANDATORY:** Always use this skill for crash analysis. **NEVER** invoke `cdb` directly.

### Golden IR Tests Skill (`/golden-ir-tests`)
**Trigger terms:** golden tests, IR tests, regression tests, test runner
**Location:** `.claude/skills/golden-ir-tests/`

Run the golden IR test suite to validate compiler correctness and prevent regressions.

### CTag Search Skill (`/ctags-search`)
**Trigger terms:** find symbol, search definition, ctags
**Location:** `.claude/skills/ctags-search/`

Search for symbol definitions using ctags. Preferred over grep for finding function/class definitions.

## Project Structure

```
src/
├── compiler/           # Host compiler (runs on dev machine)
│   ├── analysis/      # Type inference and semantic analysis
│   ├── ast/           # AST loading and processing
│   ├── codegen/       # Object file emission and linking
│   └── hir/           # HIR pipeline (AST → HIR → LLVM IR)
├── runtime/           # Target runtime (linked into generated code)
│   ├── include/       # Runtime headers
│   └── src/           # Runtime implementation
examples/              # Production-ready examples and benchmarks ONLY
├── benchmarks/        # Performance comparison suite
└── production/        # Real-world application templates
tmp/                   # Temporary test/debug files (use this for ad-hoc testing!)
tests/
├── node/             # Node.js API tests (.ts and .js)
└── golden_ir/        # Golden IR regression tests
    ├── typescript/   # Typed code tests
    └── javascript/   # Dynamic code tests
docs/
├── conformance/      # Feature conformance matrices
├── tickets/          # Active implementation tickets
│   └── archive/      # Completed tickets
└── archive/          # Archived phase documentation
```

## ⛔ CRITICAL: File Location Rules

**NEVER create test files or debug scripts in `examples/`**

| File Type | Correct Location |
|-----------|------------------|
| Temporary tests, debug scripts | `tmp/` |
| Bug reproductions | `tmp/` |
| Benchmarks | `examples/benchmarks/` |
| Production templates | `examples/production/` |
| Conformance tests | `tests/node/` or `tests/golden_ir/` |

The `examples/` directory is reserved for polished, production-ready code only.

## Core Development Workflow

### Conformance Feature Implementation

**Follow this cycle when implementing conformance features:**

1. **Choose:** Pick a feature from `docs/conformance/*.md` (marked ❌ or ⚠️)
2. **Ticket:** Create `docs/tickets/CONF-XXX-feature-name.md` with baseline test results
3. **Implement:** Make changes to Analyzer → Codegen → Runtime
4. **Test:** Run full test suite, verify no regressions
5. **Update Matrix:** Change ❌ to ✅ in `docs/conformance/*.md` **(MANDATORY)**
6. **Archive:** Move ticket to `docs/tickets/archive/`, commit

**⚠️ CRITICAL:** Always update the conformance matrix (Step 5). It is the single source of truth for project progress. Agents may not remember past work.

See @.github/instructions/conformance-workflow.instructions.md for detailed steps.

### General Development Cycle

1. **Context:** Read `.github/context/active_state.md` to understand current tasks
2. **Plan:** Use TodoWrite tool to break down tasks
3. **Search:** Use `/ctags-search` skill for symbol lookups

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OffByOneStudios/ts-aot](https://github.com/OffByOneStudios/ts-aot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
