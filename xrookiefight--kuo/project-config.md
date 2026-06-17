---
trigger: always_on
description: This document provides guidance for humans and AI coding agents (such as Claude, Copilot, or similar tools) working on the Kuo compiler codebase. It describes the architecture, conventions, and rules that agents must follow to make safe, correct contributions.
---

# AGENTS.md - AI Agent Guidelines for Kuo

This document provides guidance for humans and AI coding agents (such as Claude, Copilot, or similar tools) working on the Kuo compiler codebase. It describes the architecture, conventions, and rules that agents must follow to make safe, correct contributions.

---

## What Is Kuo?

Kuo is a compiled programming language that transpiles to C++. The compiler is written in C++17 and follows a classic pipeline:

```
.kuo source
    │
    ▼
 Lexer          (src/lexer.cpp)       Tokenizes source text into a flat token stream
    │
    ▼
 Parser         (src/parser.cpp)      Builds an AST from the token stream
    │
    ▼
 Code Generator (src/codegen.cpp)     Walks the AST and emits C++ source
    │
    ▼
 C++ Compiler   (g++ / clang++)       Produces a native binary
```

Each stage is independent. Agents should understand which stage a task belongs to before making changes.

---

## Core Invariants

These invariants must be preserved by any agent-authored change:

1. **AST nodes are owned by `std::unique_ptr`.** Never use raw `new` or raw owning pointers for AST nodes. Pass ownership via `std::move`.

2. **The pipeline is one-directional.** The lexer does not call the parser; the parser does not call the codegen. Data flows forward only via return values.

3. **`ast.h` is the shared contract.** The parser produces nodes defined in `ast.h`. The codegen consumes them. If you add a new AST node, you must update both the parser (to produce it) and the codegen (to consume it). An unhandled node type in `genStmt()` or `genExpr()` will throw at runtime - this is intentional.

4. **Zero warnings policy.** All code must compile cleanly under `g++ -std=c++17 -Wall -Wextra`. Do not suppress warnings with pragmas or casts unless absolutely necessary and documented.

5. **The `examples/` directory is the test suite.** After any change, `make test` must pass without modification to any existing example. If a change alters output intentionally, update the example and document why.

---

## How to Add a New Language Feature

Adding a feature (e.g., a `break` statement, a new operator, a new type) always touches at least three files. Follow this checklist:

### Step 1 - Token (if needed)
If the feature requires a new keyword or operator, add it to `include/token.h`:
- Add the variant to `TokenType`
- Add the string mapping in `tokenTypeName()`
- Add the keyword to the `KEYWORDS` map in `src/lexer.cpp`

### Step 2 - AST Node (if needed)
If the feature introduces new syntax structure, add a node to `include/ast.h`:
- For statements, inherit from `Stmt`
- For expressions, inherit from `Expr`
- Keep node structs simple: store only what the codegen needs
- Use `ExprPtr` / `StmtPtr` (`std::unique_ptr<Expr/Stmt>`) for child ownership

### Step 3 - Parser
Add a parsing method to `src/parser.cpp`:
- Dispatch from `parseStatement()` or `parseExpr()` based on where the feature appears syntactically
- Throw `ParseError` (not `std::runtime_error`) on unexpected tokens so line/col info is preserved
- Consume tokens with `consume()`, validate with `expect()`, test with `check()` and `match()`

### Step 4 - Code Generator
Add a generation method to `src/codegen.cpp`:
- Add a `dynamic_cast` branch in `genStmt()` or `genExpr()`
- Emit valid C++17. When in doubt, check what `./kuo --emit-cpp` produces and verify it compiles with `g++ -std=c++17`
- Update `inferType()` if the new expression has a deterministic type

### Step 5 - Example & Test
Add or update a `.kuo` file in `examples/` that exercises the new feature. Run `make test` to confirm all examples pass.

---

## How the Codegen Scope System Works

`CodeGen` maintains a scope stack (`std::vector<std::unordered_map<std::string, KuoType>>`):

- `pushScope()` / `popScope()` - called at block entry/exit
- `declareVar(name, type)` - registers a variable in the current scope
- `lookupVar(name)` - walks the stack from innermost to outermost

This is used by `inferType()` to determine the C++ type of an `IdentExpr`. When adding a new statement that introduces a variable, always call `declareVar` after resolving the type, before generating any sub-expressions that might reference it.

Function return types are collected in `funcReturnTypes` during a first-pass scan in `generate()`, so recursive function calls resolve correctly.

---

## String Concatenation Rules

Kuo allows `+` between strings and any other primitive type. The codegen in `genBinary()` handles this by wrapping non-string operands:

- `int` / `float` → `std::to_string(...)`
- `bool` → `(std::string(... ? "true" : "false"))`
- `string` → used as-is

If you add a new type, update this logic in `genBinary()`.

---

## Error Handling Conventions

| Location   | Exception Type | Fields             |
|------------|----------------|--------------------|
| Lexer      | `std::runtime_error` | message with line info embedded |
| Parser     | `ParseError`   | `message`, `line`, `col` |
| CodeGen    | `std::runtime_error` | message only |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xRookieFight/kuo](https://github.com/xRookieFight/kuo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
