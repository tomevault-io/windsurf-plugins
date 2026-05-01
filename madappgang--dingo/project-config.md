---
trigger: always_on
description: **We have FAILED to follow this rule MULTIPLE TIMES. This is non-negotiable.**
---

# Claude AI Agent Instructions - Dingo Project

## 🚨🚨🚨 STOP: READ THIS BEFORE ANY IMPLEMENTATION 🚨🚨🚨

**We have FAILED to follow this rule MULTIPLE TIMES. This is non-negotiable.**

### The Architectural Principle (understand this FIRST)

> **Position information flows through the token system, NEVER through byte arithmetic.**

The rule isn't "avoid these 5 functions". The rule is: ALL position tracking must use `token.Pos` and `token.FileSet`, not byte offsets or string scanning.

### WHY This Rule Exists

Byte offsets are **FRAGILE** - they shift when go/printer reformats code:
```
Before go/printer: "x := foo()\n"    (offset 5 = 'f')
After go/printer:  "x := foo()\n"    (tabs→spaces, offset 5 = different char!)
```

`token.Pos` is **STABLE** because it's a logical reference into a FileSet, not a physical byte position. The Go toolchain uses this throughout for a reason.

### ❌ FORBIDDEN Patterns (ALL of these, not just the listed functions):

**Direct byte/string manipulation:**
- `bytes.Index()`, `bytes.HasPrefix()`, `bytes.Contains()`
- `strings.Index()`, `strings.Contains()`, `strings.Split(string(src), ...)`
- `regexp.MustCompile()`, `regexp.Match()`, `regexp.Find*()`

**Character scanning loops:**
- `for i := 0; i < len(src); i++ { if src[i] == '?' ... }`
- Any loop that counts newlines to calculate line numbers

**Byte slice extraction from source:**
- `src[start:end]` for extracting source content by offset
- `string(src[loc.Start:loc.End])`

**Output scanning:**
- Scanning generated Go code to extract position info
- Regex/string matching on output to find `//line` directives

### ✅ REQUIRED Approaches (the ONLY correct ways):

**For Dingo source positions:**
```go
// Positions come from AST nodes (parser already tracked them)
pos := dingoNode.Pos()  // This is token.Pos
position := dingoFset.Position(pos)  // Line, column
```

**For Go source analysis (e.g., finding //line directives):**
```go
// Use go/scanner - it tokenizes with proper position tracking
fset := token.NewFileSet()
file := fset.AddFile("", fset.Base(), len(goSource))
var s scanner.Scanner
s.Init(file, goSource, nil, scanner.ScanComments)

for {
    pos, tok, lit := s.Scan()
    if tok == token.EOF { break }
    if tok == token.COMMENT && strings.HasPrefix(lit, "//line ") {
        goLine := fset.Position(pos).Line  // Token-based!
    }
}
```

**For byte offset → line:col conversion:**
```go
// Use token.FileSet, NOT manual newline counting
fset := token.NewFileSet()
file := fset.AddFile("", fset.Base(), len(src))
file.SetLinesForContent(src)  // FileSet scans internally
position := fset.Position(file.Pos(offset))
return position.Line, position.Column
```

**Key insight: Track during generation, don't scan output**
```go
// ❌ WRONG: Scan output after generation
lineMappings := extractFromOutput(generatedCode)

// ✅ RIGHT: Track during generation
for _, transform := range transforms {
    emit(transform.Code)
    mappings = append(mappings, LineMapping{
        DingoLine: transform.SourceLine,  // Already known from AST!
    })
}
```

### Pipeline Architecture
```
Source → pkg/tokenizer/ → []Token → pkg/parser/ → AST → pkg/ast/*_codegen.go
         ↑                                                 ↑
   ONLY place that                                   ONLY accepts
   reads raw bytes                                   AST nodes
```

### Pre-Implementation Checklist

Before writing ANY transform code, ask:
- [ ] Am I about to read raw bytes? → Only `pkg/tokenizer/` should
- [ ] Am I about to scan for a pattern? → Use `go/scanner` instead
- [ ] Am I calculating line:col from offset? → Use `token.FileSet`
- [ ] Am I extracting info from output? → Track during generation instead
- [ ] Would this break if go/printer reformats? → It's wrong

### Before implementing ANY feature:
1. Check if parser already handles it: `pkg/parser/`
2. Check if codegen exists: `pkg/ast/*_codegen.go`
3. If adding new syntax: extend parser FIRST, then codegen from AST

### Verification (run before ANY PR):
```bash
grep -rn "bytes\.Index\|bytes\.HasPrefix\|strings\.Index\|strings\.Split.*src\|regexp\." \
  pkg/transpiler/*.go pkg/codegen/*.go pkg/ast/*_codegen.go 2>/dev/null \
  | grep -v "_test\.go" | grep -v "// OK:"
# Must return NOTHING
```

### Post-Edit Hook

A verification hook runs automatically after code changes to catch violations.
See `.claude/hooks/README.md` for details.

---

## What is Dingo?
A meta-language for Go (like TypeScript for JavaScript):
- Transpiles `.dingo` files to idiomatic `.go` files
- Provides Result/Option types, pattern matching, error propagation (`?`)
- 100% Go ecosystem compatibility via gopls proxy for IDE support

## Project Structure
```
cmd/dingo/          # CLI entry point
pkg/
├── ast/            # Code generators (*_codegen.go) - FROM AST ONLY
├── parser/         # Dingo parser (Pratt-based) - PRODUCES AST
├── tokenizer/      # Tokenizer - ONLY place reading raw bytes
├── goparser/       # Go parser wrapper + transforms
├── feature/        # Pluggable feature system
├── transpiler/     # Main pipeline
└── typechecker/    # go/types integration
tests/golden/       # Golden test files
examples/           # Example .dingo files
```

## Architecture

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MadAppGang/dingo](https://github.com/MadAppGang/dingo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
