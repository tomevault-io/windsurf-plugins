---
trigger: always_on
description: You are working on **Papyrus Compiler** — an open-source compiler for the Papyrus scripting language (Skyrim SE/AE). The compiler is written in the **V programming language** (vlang.io). It compiles `.psc` (Papyrus Source Code) files into `.pex` (Papyrus Executable) binary files — the same format as Bethesda's official Creation Kit compiler.
---

# AGENTS.md — System Prompt for AI Assistants

You are working on **Papyrus Compiler** — an open-source compiler for the Papyrus scripting language (Skyrim SE/AE). The compiler is written in the **V programming language** (vlang.io). It compiles `.psc` (Papyrus Source Code) files into `.pex` (Papyrus Executable) binary files — the same format as Bethesda's official Creation Kit compiler.

---

## Project Structure

```
compiler.v              — Entry point: CLI dispatch (compile, read, disassembly, dump, help, version)
fast.v                  — Benchmarking harness for performance tracking across commits
v.mod                   — V module metadata (version, dependencies)
run.bat                 — Quick-launch script for compilation

modules/
├── pref/               — CLI argument parsing, Preferences struct
├── papyrus/
│   ├── token/          — Token kinds (enum Kind), Position struct
│   ├── scanner/        — Lexer: .psc text → token stream
│   ├── ast/            — AST nodes, type system, symbol table (Table), scopes
│   ├── parser/         — Recursive-descent parser: tokens → AST
│   ├── checker/        — Semantic analysis: type checking, cast validation, scope resolution
│   ├── errors/         — Error/Warning structs, predefined error message constants
│   └── util/           — Helpers: BOM handling, char classification, error formatting
├── pex/                — PEX binary format: data structures, reader, writer, opcodes, dump
├── gen/
│   ├── gen_pex/        — Code generator: AST → PEX bytecode instructions
│   └── ts_binding/     — TypeScript binding generator (secondary feature)
├── builder/            — Orchestrator: drives the full compile pipeline, caching, stats
└── tests/              — Test suite (V's built-in test framework)
```

### Directories that should NOT be modified:
- `modules/tests/*Sources/` — Third-party Skyrim mod source files used as test fixtures. Do not edit.
- `modules/tests/psc_deps/` — 83 Skyrim base class header stubs (Actor.psc, Form.psc, etc.) used as dependencies for tests. Do not edit unless specifically adding new base class stubs.
- `test-files/` — Output directory for compiled test artifacts (.pex files). Not source code.
- `bin/` — Build output directory for the compiler binary.

---

## Compilation Pipeline

The compiler processes files through these stages in order:

```
.psc source → Scanner → Parser → Checker → Gen (gen_pex) → PEX Writer → .pex binary
```

1. **Scanning** (`modules/papyrus/scanner/`) — Character-by-character lexer. Handles `;` line comments, `{block}` comments, `;/ multi-line /;` comments. Case-insensitive keywords. Line continuation with `\`.
2. **Parsing** (`modules/papyrus/parser/`) — Recursive-descent parser producing `[]&ast.File`. Split into: `parser.v` (main + statements), `expr.v` (expressions with precedence climbing), `fn.v` (function/event declarations), `type.v` (type parsing). **Selective header loading**: unknown types are pushed onto `table.deps` stack, resolved by the builder iteratively.
3. **Checking** (`modules/papyrus/checker/`) — Type checking, autocast validation, scope resolution, method resolution via inheritance chains. Split into: `checker.v` (core), `checker_stmt.v` (statements), `checker_expr.v` (expressions).
4. **Code Generation** (`modules/gen/gen_pex/`) — Generates PEX bytecode from AST. Manages temp variables (`::temp0`, `::temp1`, ...), string interning, control flow jump patching. Split into: `gen.v` (main), `gen_stmt.v` (statements), `gen_expr.v` (expressions).
5. **PEX Writing** (`modules/pex/writer.v`) — Big-endian binary serialization. Generic `write[T]()` with compile-time type dispatch.

---

## Key Data Structures

### Type System (`modules/papyrus/ast/types.v`)
- `Type = int` — Index into `Table.types[]`
- Built-in type indices: 1=None, 2=Int, 3=Float, 4=String, 5=Bool, 6=Array, 7-10=typed arrays (String[], Int[], Float[], Bool[])
- `TypeSymbol` — Holds: kind, parent_idx, methods, props, states, vars
- Placeholder types used for forward references, resolved during dependency loading

### Symbol Table (`modules/papyrus/ast/table.v`)
- `Table` — Central registry: `types[]TypeSymbol`, `type_idxs map[string]int`, `fns map[string]Fn`, `deps Stack[string]`
- **All lookups are case-insensitive** (`.to_lower()` on keys)
- Functions keyed as `"objname.fnname"`

### AST Nodes (`modules/papyrus/ast/ast.v`, `expr.v`)
- Top-level: `TopStmt = ScriptDecl | FnDecl | Comment | PropertyDecl | VarDecl | StateDecl`
- Statements: `Stmt = Return | If | While | ExprStmt | AssignStmt | VarDecl | Comment`
- Expressions: `Expr = InfixExpr | IntegerLiteral | FloatLiteral | BoolLiteral | StringLiteral | Ident | CallExpr | SelectorExpr | IndexExpr | ParExpr | PrefixExpr | EmptyExpr | ArrayInit | NoneLiteral | CastExpr`

---

## V Language Conventions Used in This Project

### Naming
- Structs: `PascalCase`
- Functions/methods: `snake_case`
- Enum variants: `snake_case` with `.` prefix (e.g., `.key_if`)
- Constants: `snake_case`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [russo-2025/papyrus-compiler](https://github.com/russo-2025/papyrus-compiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
