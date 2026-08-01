---
trigger: always_on
description: Use when: interactive web UIs, real-time dashboards, forms, admin panels.
---

# CLAUDE.md — Sky Language Project

This is a [Sky](https://github.com/anzellai/sky) project. Sky is a pure functional language inspired by Elm, compiling to Go. The compiler is fully self-hosted (written in Sky, ~6MB native binary, zero Node/TypeScript dependencies).

**Core principle: if it compiles, it works.** All side effects flow through `Task`. No runtime panics, no nil leakage, no partial bindings.

## Quick Reference

```bash
sky init [name]           # Create a new Sky project (sky.toml, src/Main.sky, .gitignore, CLAUDE.md)
sky build src/Main.sky    # Compile to Go binary (output: sky-out/app)
sky run src/Main.sky      # Build and run
sky check src/Main.sky    # Type-check without compiling (cross-module ADT + alias resolution)
sky fmt src/Main.sky      # Format code (Elm-style: 4-space indent, leading commas)
sky add <package>         # Add dependency + generate bindings + update sky.toml
sky remove <package>      # Remove dependency from sky.toml + clean cache
sky install               # Install all deps + auto-generate missing bindings
sky update                # Update sky.toml dependencies to latest
sky upgrade               # Self-upgrade Sky compiler to latest release
sky lsp                   # Start Language Server
sky clean                 # Remove build artifacts
sky --version             # Show version
```

## Language Syntax

```elm
module Main exposing (main)

import Sky.Core.Prelude exposing (..)    -- auto-imported: Result, Maybe, identity, errorToString
import Sky.Core.String as String
import Sky.Core.List as List
import Sky.Core.Dict as Dict
import Std.Log exposing (println)

-- Type annotations are optional (Hindley-Milner inference)
greet : String -> String
greet name =
    "Hello, " ++ name

-- Algebraic data types
type Shape
    = Circle Float
    | Rectangle Float Float

-- Records (type aliases)
type alias Point = { x : Int, y : Int }

-- Pattern matching (exhaustiveness checked by compiler)
area : Shape -> Float
area shape =
    case shape of
        Circle r -> 3.14 * r * r
        Rectangle w h -> w * h

-- Let-in expressions
main =
    let
        p = { x = 10, y = 20 }
        updated = { p | x = 99 }     -- immutable record update
        items = [1, 2, 3]
            |> List.map (\x -> x * 2)  -- pipeline operator
            |> List.filter (\x -> x > 3)
    in
    println "Result:" (String.fromInt updated.x)
```

### Types

`Int`, `Float`, `String`, `Bool`, `Char`, `Unit` (`()`), `List a`, `Maybe a` (`Just a | Nothing`), `Result err ok` (`Ok ok | Err err`), `Dict k v`, tuples `(a, b)`, records `{ field : Type }`

### Operators

`++` (concat), `|>` `<|` (pipe), `>>` `<<` (compose), `==` `!=` `/=` `<` `>` `<=` `>=`, `&&` `||`, `+` `-` `*` `/` `//` `%`, `::` (cons)

Note: `/=` is Elm-compatible not-equal (alias for `!=`). `//` is integer division (always returns `Int`). Both forms are supported.

### Multiline Strings

Triple-quoted strings preserve newlines. Interpolation uses double braces `{{expr}}`:

```elm
html =
    """<div class="card">
    <h1>{{title}}</h1>
    <p>{{description}}</p>
</div>"""

sql =
    """CREATE TABLE IF NOT EXISTS users (
        id TEXT PRIMARY KEY,
        name TEXT NOT NULL
    )"""
```

Single braces `{` are literal — safe for JavaScript, CSS, JSON, SQL. Interpolation expressions support identifiers, field access, qualified names, and function calls.

### Patterns

Literals, constructors (`Just x`, `Ok v`, `Err e`), tuples `(a, b)`, lists `[]`, `[x]`, `x :: xs`, wildcards `_`, as-patterns `Just x as original`, nested `Ok (Just x)`

### Record Patterns

```elm
-- Record patterns (destructuring)
case user of
    { name, age } -> name ++ " is " ++ String.fromInt age

-- In function params
greet { name } = "Hello, " ++ name

-- In let bindings
let { x, y } = point in x + y
```

## Task — Effect Boundary

All side effects (IO, HTTP, file access) flow through `Task`. Tasks are lazy — they only execute when `perform` is called. Panics are caught and converted to `Err`.

```elm
import Sky.Core.Task as Task

-- Create and compose Tasks
pipeline =
    Task.succeed "Sky"
        |> Task.andThen (\name -> Task.succeed ("Hello, " ++ name ++ "!"))
        |> Task.map (\msg -> msg ++ " Pure and reliable.")

-- Execute at the boundary
result = Task.perform pipeline
-- result : Result String String
```

**Task API:**
- `Task.succeed : a -> Task err a`
- `Task.fail : err -> Task err a`
- `Task.map : (a -> b) -> Task err a -> Task err b`
- `Task.andThen : (a -> Task err b) -> Task err a -> Task err b`
- `Task.perform : Task err a -> Result err a`
- `Task.sequence : List (Task err a) -> Task err (List a)` -- run sequentially
- `Task.parallel : List (Task err a) -> Task err (List a)` -- run concurrently (goroutines)
- `Task.lazy : (() -> a) -> Task err a` -- defer computation until executed
- `Task.map2/3/4/5` -- combine N independent Tasks (sequential, like Result.mapN) (v0.7.25+)
- `Task.andMap : Task e a -> Task e (a -> b) -> Task e b` -- pipeline-style applicative (v0.7.25+)

**Concurrency:**

`Task.parallel` runs tasks concurrently using Go goroutines. Results are collected in order; the first error short-circuits.

```elm
-- Parallel HTTP requests (total time = slowest request, not sum)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anzellai/sky](https://github.com/anzellai/sky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
