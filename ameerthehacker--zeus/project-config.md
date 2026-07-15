---
trigger: always_on
description: Zeus is a compiled language that emits native binaries via LLVM. The pipeline is:
---

# Zeus Compiler — Developer Reference

## Quick orientation

Zeus is a compiled language that emits native binaries via LLVM. The pipeline is:

```
.zs source  →  Lexer  →  Parser  →  IR gen  →  Type check  →  Lowering  →  Codegen  →  LLVM  →  binary
```

The runtime is written in Zig (`runtime/`). Build tags: `-tags llvm19` are required for all `go` commands.

## Repo layout

```
cmd/                  CLI (cobra commands: build, fmt, lsp, ...)
internal/
  token/              TokenType enum + Span/Position types
  lexer/              Lexer — produces []Token from source text; comments captured off-stream via Comments()
  ast/                AST nodes + ExprVisitor[T] interface
  parser/             Pratt parser (precedence climbing)
  ir/
    ir.go             IR generator — walks AST, emits HIR via IRBuilder
    builder.go        IRBuilder — creates BasicBlocks and Instr lists
    instr.go          InstrType enum, VarDecl, input/output structs
    tc.go             Type-checker driver (pass list, context management, shared utilities)
    tc_type_check.go  TypeCheckingPass — type validation, implicit casts, safety-net resolution
    tc_unused.go      UnusedWarningPass
    tc_undefined.go   UndefinedTypeCheckPass
    type_inference.go TypeInferencePass — NEW_OBJ + OBJECT_PROPERTY_ACCESS bidirectional sync
    ir_type_infer.go  inferFunctionEnv — AST pre-scan for local var types + return type inference
    ir_passes.go      DeclCheckPass (pre-IR stub registration) + other pre-IR passes
    lowering.go       HIR → LIR lowering (e.g. array indexing → method calls)
  codegen/
    codegen.go        LIR → LLVM IR via tinygo.org/x/go-llvm
    llvm_type.go      Zeus type → LLVM type conversion
  zeus_value/         Value and ValueType interfaces, concrete types (IntType, FloatType, ...)
  zeus_compiler/      Orchestrates all passes in order
  formatter/          `zeus fmt` — Prettier-style formatter (doc.go IR, printer.go, formatter.go). See wiki/formatter.md
  lsp/                Language-server protocol implementation
test/
  e2e/                End-to-end tests; compiled via the zeus binary
  parser/             Parser unit tests (AST structure)
  formatter/          Formatter golden + idempotency/parse-stability tests
  ir/                 HIR/LIR blackbox tests
  lexer/              Lexer unit tests
```

## How to run tests

```bash
# All tests (requires LLVM headers on PATH)
go test -tags llvm19 ./test/...

# Just e2e (builds the zeus binary first)
go test -tags llvm19 ./test/e2e/... -v -count=1

# Parser/lexer/IR tests (no LLVM needed)
go test ./test/parser/... ./test/lexer/... ./test/ir/...

# Compile a playground file (needs cmake build + zig runtime)
make compile file=hello
make run file=hello
```

## Adding a new operator — complete checklist

### Step 1 — Token (`internal/token/token.go`)

Add the new `TokenType` constant inside the `operator_beg`/`operator_end` sentinels and add its `String()` case.

### Step 2 — Lexer (`internal/lexer/lexer.go`)

Add a `case char == 'X':` branch in the main `Lex()` switch. For multi-character tokens, use `l.matchNextRune('Y')` to peek ahead. Emit with `l.pushToken(token.NewToken(...))`.

### Step 3 — Parser (`internal/parser/parser.go`)

**Precedence table** — add the new token to `BinaryOperatorPrecedence` with the right integer:

| Prec | Operators |
|------|-----------|
| 20 | `.` `[]` member access / indexing |
| 19 | `()` function call, `new` |
| 18 | postfix `++` `--` |
| 17 | unary `-` `!` `~` prefix `++` `--` (`UnaryOperatorPrecedence`) |
| 16 | `**` right-assoc |
| 15 | `*` `/` `%` |
| 14 | `+` `-` |
| 13 | `<<` `>>` |
| 12 | `<` `<=` `>` `>=` |
| 11 | `==` `!=` |
| 10 | `&` bitwise AND |
| 9 | `^` bitwise XOR |
| 8 | `\|` bitwise OR |
| 7 | `&&` logical AND |
| 6 | `\|\|` logical OR |
| 3 | `?:` ternary |
| 1 | `=` `+=` `-=` `*=` `/=` `%=` `**=` assignments |

**Right-associative** — add to `RightAssociativeOperators` if needed (assignments, `**` are right-assoc).

**Parselet** — add to `infixParselets` (binary) or `prefixParselets` (unary). Most binary operators use the shared `binaryOperatorParseLet`. Special operators (ternary, function call, indexing) get their own closure.

### Step 4 — AST node (only for non-binary operators)

If the operator requires a new AST shape (e.g. `TernaryExprNode` with three children), add the node in `internal/ast/expr.go` and add the corresponding `VisitXxx(*XxxNode) T` method to the `ExprVisitor[T]` interface. There are two implementations to update: `*IRModule` in `ir.go` (full IR emission) and `astWalker` in `ir/closure.go` (closure analysis — add a stub that walks any sub-expressions via `w.walkExpr`).

### Step 5 — IR instruction (`internal/ir/instr.go`)

Add a new `InstrTypeXxx` constant in the `const (...)` block and a `String()` case.

### Step 6 — IR generation (`internal/ir/ir.go`)

Add a case in `VisitBinaryExpr` (or `VisitUnaryExpr`, or a new `VisitTernaryExpr` method). Use `g.irBuilder.BuildBinaryOp` / `BuildUnaryOp`. For control-flow operators (short-circuit, ternary), use `BuildSuccessorBlock`, `BuildCondJmp`, `BuildJmp`, `BuildStore`, `BuildLoad` — see `emitShortCircuitAnd` as the pattern.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ameerthehacker/zeus](https://github.com/ameerthehacker/zeus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
