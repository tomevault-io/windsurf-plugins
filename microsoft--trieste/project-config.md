---
trigger: always_on
description: Trieste is a **header-only C++20 term rewriting system** for rapidly prototyping programming languages. It provides three C++ embedded DSLs:
---

# Trieste — Copilot Instructions

## What is Trieste?

Trieste is a **header-only C++20 term rewriting system** for rapidly prototyping programming languages. It provides three C++ embedded DSLs:

1. **Parsing DSL** — regex-based tokenization that produces an untyped AST from source files.
2. **Rewriting DSL** — pattern matching and term rewriting to reshape ASTs through successive passes.
3. **Well-formedness (WF) DSL** — declarative specifications of valid AST shapes, used for validation and automatic fuzz-test generation.

Language implementation in Trieste is a pipeline: `Parse → Pass₁ → Pass₂ → ... → Passₙ → Output`, where each pass rewrites the AST from one well-defined shape to another.

## Core Concepts

### Tokens (`include/trieste/token.h`)

Tokens are the type system for AST nodes. Each is defined as an `inline const auto` at namespace scope:

```cpp
inline const auto Ident = TokenDef("infix-ident", flag::print);
inline const auto Calculation = TokenDef("infix-calculation", flag::symtab | flag::defbeforeuse);
```

Token names are prefixed with the language name (e.g. `infix-`, `json-`) to avoid collisions. Flags control node behavior:
- `flag::print` — display source text when printing the AST
- `flag::symtab` — node carries a symbol table
- `flag::defbeforeuse` — symbols must be defined before use
- `flag::lookup` / `flag::lookdown` — symbol visibility directions
- `flag::shadowing` — stops lookup ascending to parent scopes

Built-in tokens include `Top`, `File`, `Group`, `Error`, `ErrorMsg`, `ErrorAst`, `Lift`, `Seq`, `NoChange`.

### AST Nodes (`include/trieste/ast.h`)

`Node` is an intrusive reference-counted pointer to `NodeDef`. Each node has a Token type, a Location (source position), a parent pointer, children (`std::vector<Node>`), and an optional symbol table. The `<<` operator builds trees fluently: `Assign << ident << expr`.

### Parsing (`include/trieste/parse.h`)

The `Parse` class implements a mode-based regex tokenizer. Rules are defined per mode as `regex >> lambda`:

```cpp
Parse p(depth::file, wf_parser);
p("start", {
    "[[:blank:]]+" >> [](auto&) {},
    R"([[:digit:]]+\b)" >> [](auto& m) { m.add(Int); },
});
```

The `Make` object provides `m.add()`, `m.push()`, `m.pop()`, `m.seq()`, `m.term()`, `m.mode()`, and `m.error()` for building the AST during parsing.

### Rewriting (`include/trieste/rewrite.h`)

Rewrite rules use operator overloading for pattern matching: `pattern >> effect`:

```cpp
In(Expression) * (T(Ident)[Id] * T(Add) * T(Ident)[Rhs]) >>
  [](Match& _) { return Add << (Expression << _(Id)) << (Expression << _(Rhs)); };
```

Pattern combinators:
- `T(Token)` — match a node type; `T(A, B, C)` — match any of those types
- `In(Token)` — parent must be that type; `In(Token)++` — any ancestor must be that type
- `P * Q` — sequence; `P / Q` — choice
- `P[Name]` — capture node as `Name`; access via `_(Name)` or `_[Name]` (range)
- `~P` — optional; `P++` — zero or more; `!P` — negation
- `Start`, `End`, `Any` — positional/wildcard matchers

Effects return `Node` (replacement), `NoChange` (skip), or `Seq` (splice multiple nodes).

### Passes (`include/trieste/pass.h`)

A `PassDef` bundles rewrite rules with a name, WF spec, and direction:

```cpp
PassDef my_pass = {
    "pass_name",
    wf_after_this_pass,
    dir::topdown,  // or dir::bottomup, dir::once
    { /* rewrite rules */ }
};
```

Passes iterate to a fixed point unless `dir::once` is set. Additional hooks: `pre()`, `post()`, `cond()`.

### Well-formedness (`include/trieste/wf.h`)

WF specs declare valid AST shapes:

```cpp
// clang-format off
const auto wf =
    (Top <<= Calculation)
  | (Calculation <<= (Assign | Output)++)
  | (Assign <<= Ident * Expression)[Ident]
  | (Expression <<= (Add | Subtract | Ref | Int))
  ;
// clang-format on
```

Key operators: `<<=` (define shape), `|` (choice/combine), `++` (zero or more), `~` (optional), `*` (fixed ordered fields), `[Token]` (symbol binding). WF specs are defined incrementally — each pass extends/overrides the previous spec using `|`.

### Reader (`include/trieste/reader.h`)

Combines a parser + ordered list of passes into a pipeline:

```cpp
Reader reader() {
    return {"json", {groups(), structure()}, parser()};
}
```

### Writer (`include/trieste/writer.h`)

Converts an AST back to output files via passes that produce `Directory`/`File`/`Contents` nodes.

### Rewriter (`include/trieste/rewriter.h`)

A lightweight wrapper for running additional pass pipelines on an already-parsed AST. Chains with `>>`: `reader >> rewriter`.

### Driver (`include/trieste/driver.h`)

Wraps a Reader with CLI11-based CLI providing `build` (parse + rewrite + output) and `test` (WF-driven fuzz testing) subcommands:

```cpp
int main(int argc, char** argv) {
    return trieste::Driver(my_reader()).run(argc, argv);
}
```

## Project Organisation

### `include/trieste/` — Header-only library

| Header | Purpose |
|--------|---------|
| `trieste.h` | Master include + pipeline operators (`>>`) |
| `token.h` | Token/TokenDef types, flags, built-in tokens |
| `ast.h` | NodeDef, symbol tables, tree manipulation |
| `source.h` | Source files, Location |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/Trieste](https://github.com/microsoft/Trieste) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
