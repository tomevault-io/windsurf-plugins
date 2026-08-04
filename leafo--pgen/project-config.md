---
trigger: always_on
description: > Last updated at commit: `c48c4881ce56e050cda9515ab3c44ac04e7d078d`
---

# CLAUDE.md

> Last updated at commit: `c48c4881ce56e050cda9515ab3c44ac04e7d078d`

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Core Commands

### Building and Testing

```bash
# Build and run tests using busted (test framework)
make busted

# Run the same suite against the pure Lua code generation target
make busted-lua
```

## Project Architecture

pgen is a Lua parser generator that takes LPEG-like pattern definitions and generates Lua modules for parsing strings. The primary target generates C code compiled to a shared object; a second target generates a self-contained pure Lua module for environments without a C compiler.

### Directory Structure

```
pgen/
├── pgen.lua              # Core API module
├── pgen_cli.lua          # CLI tool
├── Makefile              # Build automation
├── pgen/                 # Core implementation modules
│   ├── generator.lua     # C code generation
│   ├── generator_lua.lua # Pure Lua code generation
│   ├── codegen_common.lua # Target-independent collection passes/templates
│   ├── optimize.lua      # Grammar optimizations (trie, flattening)
│   ├── types.lua         # Type constants (P=1, R=2, etc.)
│   └── visitor.lua       # AST visitor pattern for traversal/transformation
├── examples/             # Example grammars (calc, JSON, numbers, Teal)
└── spec/                 # Test suite (busted framework)
    ├── *_spec.lua        # Test files
    └── parsers/          # Test grammars and their generated C/so files
```

pgen is published as a rock (`pgen-dev-1.rockspec`); `luarocks make` installs
the working tree locally, including the `pgen` CLI binary.

The largest real-world grammar built on pgen is the MoonScript parser, which
lives in its own repository (https://github.com/leafo/moonscript-parser,
locally at `../../moon/moonscript-parser`). Its checked-in C parser is
generated with the installed pgen rock, so after generator changes run
`make local` here followed by `make generate && make test` there.

### Key Components

1. **pgen.lua**: Core module with pattern constructors and compilation functions
2. **pgen/generator.lua**: C code generator that transforms grammar into parser code
3. **pgen/generator_lua.lua**: Lua code generator with the same feature set and semantics as the C target
4. **pgen/codegen_common.lua**: Collection passes and template helpers shared by both generators
5. **pgen/optimize.lua**: Grammar optimizer (trie optimization for literal alternatives, choice flattening)
6. **pgen/visitor.lua**: Generic visitor pattern for AST traversal and transformation
7. **pgen/types.lua**: Type constants for pattern types
8. **pgen_cli.lua**: Command-line interface for the generator

### Workflow

1. Define a grammar using pattern constructors (P, R, S, V, etc.)
2. Compile the grammar to C code with `pgen.compile()`
3. Compile the C code to a shared object (.so file)
4. Load the shared object as a Lua module
5. Use the module's `parse()` function to parse input strings

For development/testing, use `pgen.require()` to dynamically compile and load grammars:

```lua
local pgen = require("pgen")
local parser = pgen.require("grammar.module") --> uses same search path as require()
parser.parse("input string")
```

### Lua Target

Passing `target = "lua"` to `pgen.compile()` or `pgen.require()` generates a
self-contained pure Lua module instead of C, for environments without a C
compiler. The generated file has no dependencies, runs on Lua 5.1+ and
LuaJIT regardless of which Lua version ran the generator, and has the same
`parse()` contract and semantics as the C target (captures, Cmt/Cfn,
labeled failures, indenters, furthest-failure positions, `pgen_errors`
messages, memoization). `pgen.require` with the Lua target loads the
generated chunk directly — no compiler, no temp files.

```lua
local parser = pgen.require("grammar.module", {target = "lua"})
```

Setting `PGEN_TARGET=lua` in the environment switches `pgen.require`'s
default target, which is how `make busted-lua` runs the whole test suite
against the Lua backend.

Performance: roughly 10-30x slower than the C parser under PUC Lua.
LuaJIT's interpreter (`-joff`) runs the generated parsers about 2x faster
than PUC Lua; the trace compiler helps small parsers (~3x over the
interpreter) but can thrash on very large ones (the MoonScript-scale
grammar runs 10x slower with tracing on than off), so benchmark before
choosing for big grammars.

### Pattern Types

- `P(string)` - Match literal string
- `P(n)` - Match exactly n characters (or fail if n < 0 and that many chars remain)
- `R(start, end)` - Match character range
- `S(set)` - Match character in set
- `V(rule)` - Reference another rule
- `C(patt)` - Capture matched text
- `Ct(patt)` - Capture into table (collects all inner captures)
- `Cp()` - Capture current position without consuming input
- `Cc(value)` - Capture constant value
- `L(patt)` - Lookahead (match without consuming input)
- `Cg(patt, name)` - Capture group (creates named field in parent Ct)
- `Cn(patt, n)` - Numbered capture (select nth capture from inner pattern)
- `Cmb(name)` - Match backreference: matches the same text captured by `Cg(patt, name)`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leafo/pgen](https://github.com/leafo/pgen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-01 -->
