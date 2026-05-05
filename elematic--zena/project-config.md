---
trigger: always_on
description: This document guides AI agents working on the Zena project. For a complete
---

# Zena Project Instructions

This document guides AI agents working on the Zena project. For a complete
language description, see `docs/language-reference.md`. For completed features
and planned work, see `PLAN.md`.

## Project Overview

Zena is a statically typed language targeting WebAssembly GC. Think of it as a
mashup of **TypeScript** (type syntax, arrow functions, modules),
**Dart** (constructors with initializer lists, `this.` params, mixins),
**Scala** (sealed class hierarchies, case classes, pattern matching,
expression orientation), and **Swift** (immutability by default, `var`/`let`
field modifiers, no `++`/`--`, compound assignment `+=` instead). It has a
sound type system with no implicit coercion.

### Language at a Glance

```zena
// Variables: let = immutable, var = mutable
let x = 42;
var y = 'hello';

// Functions: arrow syntax only
let add = (a: i32, b: i32) => a + b;
let greet = (name: String) => {
  return 'Hello, ' + name;
};

// Classes: fields immutable by default, Dart-style constructors
class Point {
  x: f64;            // immutable (default)
  y: f64;            // immutable
  new(this.x, this.y);
}

class Counter {
  var(#count) count: i32 = 0;  // public getter, private setter
  increment() { this.#count += 1; }
}

// Case classes: concise data types with auto-generated ==, hashCode
class Pair<A, B>(first: A, second: B)

// Sealed classes: closed hierarchies for exhaustive matching
sealed class Expr {
  case Lit(value: i32)
  case Add(left: Expr, right: Expr)
}

// Pattern matching with match() expressions
let eval = (e: Expr): i32 => match (e) {
  case Lit {value}: value
  case Add {left, right}: eval(left) + eval(right)
};

// Enums: nominal wrapper types
enum Color { Red, Green, Blue }

// Type aliases
type Point = {x: f64, y: f64};

// Records and tuples: lightweight immutable data
let origin: Point = {x: 0.0, y: 0.0};
let pair = (1, 'hello');
let {x, y} = origin;  // destructuring

// Pattern matching, for-in, if-let
for (let item in items) { ... }
if (let Some {value} = maybeVal) { ... }

// Modules: ES-style imports/exports
import {Map} from 'zena:collections';
export let main = () => 0;
```

Key things that differ from TypeScript:

- **No `function` keyword** — arrow functions only.
- **No `const`** — use `let` (immutable) and `var` (mutable).
- **Class fields are immutable by default** — use `var` to make mutable.
- **Dart-style constructors** — initializer lists (`: x = x, y = y`), `this.`
  params, semicolon bodies.
- **`String` not `string`** — capital S (it's a class, not a primitive alias).
- for/in loops iterator on iterables and iterators: `for (let item of items)`.
  They are like for/of loops in TypeScript.
- **No `++`/`--`** — use `+= 1` instead.
- **Sound type system** — no `any` escape hatch (well, `any` exists but requires
  explicit casts back).
- **`match` expressions** with exhaustiveness checking.
- **Sealed classes** for sum types, not TypeScript discriminated unions.

## Two Compilers

The project has **two compiler implementations**:

1. **Bootstrap compiler** (`packages/compiler`): Written in TypeScript. Mostly
   working. Package name: `@zena-lang/compiler`.
2. **Self-hosted compiler** (`packages/zena-compiler`): Written in Zena.
   Partially implemented — currently has lexer, parser, and early type checker
   stages. Package name: `@zena-lang/zena-compiler`. See
   `docs/design/self-hosted-compiler.md` for the architecture.

Both compilers should pass the same **portable tests** in `tests/language/`.

## Portable Tests

Tests in `tests/language/` are compiler-agnostic `.zena` files with comment
directives. They're organized into three categories:

- **`syntax/`** — Parser tests. Directive: `// @target: module|statement|expression`
- **`semantics/`** — Type checker tests. Directive: `// @mode: check` with `// @error:` on error lines.
- **`execution/`** — Codegen tests. Directive: `// @mode: run` with `// @result:` for expected return value.

When fixing bugs or adding features, prefer adding portable tests here over
TypeScript-only tests in `packages/compiler/src/test/`.

## Project Structure

This project is an **npm monorepo** managed with **Wireit**.

- **`packages/compiler`**: Bootstrap compiler (`@zena-lang/compiler`).
- **`packages/zena-compiler`**: Self-hosted compiler (`@zena-lang/zena-compiler`).
- **`packages/stdlib`**: Standard library (`@zena-lang/stdlib`).
- **`packages/cli`**: CLI tool (`@zena-lang/cli`).
- **`packages/runtime`**: JS runtime helpers.
- **`tests/language/`**: Portable tests (shared across compilers).
- **`docs/language-reference.md`**: Official language reference.
- **`docs/design/`**: Design documents for complex features.
- **`PLAN.md`**: Completed and planned work.

# Interaction Guidelines

- **Conversational Requirement**: You MUST explain your plan in plain English
  BEFORE generating code or editing files. Do not act silently.
- **Tool Usage**:
  - NEVER create temporary files or shell scripts to edit code.
  - ALWAYS use the provided VS Code text editing tools to modify files
    directly.
- **Clarification Protocol**: If a request is ambiguous or lacks context, you
  MUST ask a clarifying question. Do not guess.
- **Task Management**: Use the Todo List tool (`manage_todo_list`) to track

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elematic/zena](https://github.com/elematic/zena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
