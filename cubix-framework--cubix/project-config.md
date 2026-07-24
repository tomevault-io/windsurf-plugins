---
trigger: always_on
description: The Cubix framework for multi-language transformation.
---

# The Cubix framework

The Cubix framework for multi-language transformation.

## Build commands with [devenv](https://devenv.sh/) (recommended)

```bash
devenv                                 # Enter dev shell with GHC, cabal, HLS
cabal build                            # Build main cubix library and executables
cabal build cubix-sui-move             # Build associated Sui Move language support library and executables
cabal build cubix-examples             # Build programs that demonstrate cubix capabilities
cabal build cubix-benchmarks:mainbench # Build benchmarks
```

Cubix is a framework for language-parametric program transformation - defining single source-to-source transformations that work across multiple programming languages. It currently supports C, Java, JavaScript, Lua, and Python. The core innovation is "incremental parametric syntax" which allows language definitions to share common components while enabling transformations to be written generically.

## Architecture Overview

### Project Structure

```
├── src/Cubix/
│   ├── Language/
│   │   ├── Parametric/     # Core parametric syntax framework
│   │   ├── C/              # C language support (deprecated, language support should be it separate package)
│   │   ├── Java/           # Java language support (deprecated, language support should be it separate package)
│   │   ├── JavaScript/     # JavaScript language support (deprecated, language support should be it separate package)
│   │   ├── Lua/            # Lua language support (deprecated, language support should be it separate package)
│   │   └── Python/         # Python language support (deprecated, language support should be it separate package)
│   ├── Transformations/    # Program transformations
│   ├── Analysis/           # Program analyses
│   ├── Sin/                # Compatibility layer
│   ├── Essentials.hs       # Beginner-friendly exports
│   └── ParsePretty.hs      # Parsing and pretty-printing
├── compdata/               # Compositional data types library
├── compstrat/              # Strategy combinators library
├── comptrans/              # Transformation utilities library
├── cubix-solidity/         # Solidity language support
├── cubix-tree-sitter/      # Generating cubix language support based on tree-sitter grammar definitions
├── cubix-examples          # Demos of cubix capabilities
├── cubix-benchmarks/       # Cubix framework benchmarks
├── cubix-sui-move/         # Generated (from tree-sitter grammar) language support for Sui Move
└── tree-sitter-sui-move/   # Contains tree-sitter grammar for sui-move
```

### Language support package structure

Supporting new language should be realised by adding new package: `cubix-{language_name}`. Within that package there should be module hierarchy like:

```
Cubix.Language.{LanguageName}
├── Modularized # Modularized AST, generated automatically by `gen-mod` executable of `cubix-tree-sitter`
├── ParsePretty # Parser for language, generated from `gen-parser` executable of `cubix-tree-sitter`
└── IPS         # Incremental Parametric Syntax, where some of the modularized nodes have been replaced by the generic ones, from the Cubix framework
```

Note that the support modules for C, Java, JavaScript, Lua and Python are bundled within the main Cubix library. We want to move away from that by splitting them out to their own separate packages. This already happened for Solidity support modules.

Also note that legacy support modules have different names. `Cubix.Language.{LanguageName}.Full` for Modularized syntax module and `Cubix.Language.{LanguageName}.Common` for Incremental Parametric Syntax module.

### Core Components of Cubix framework

**Language Fragments (`src/Cubix/Language/Parametric/Syntax/`):**
- Base syntax definitions shared across languages
- Expression, Function, VarDecl, and Functor fragments
- Built using higher-kinded data types with kind `(* -> *) -> * -> *`

**Transformation Framework (`src/Cubix/Transformations/`):**
- `Hoist/` - Code hoisting transformations
- `TAC/` - Three-address code conversion
- `Plumbing/IPT.hs` - Interprocedural plumbing transformation
- `TestCoverage.hs` - Test coverage analysis
- `Variation.hs` - Program variation generation

**Analysis Framework (`src/Cubix/Analysis/`):**
- `Call/Trivial.hs` - Basic call analysis

### Key Types and Concepts

**Signatures:** Type-level lists of language fragments (e.g., `MLuaSig`, `MJavaSig`)

**Terms:** `Term fs l` represents terms of signature `fs` with sort `l`

**Language Fragments:** Defined using GADTs with explicit sort annotations

**Sort Injections:** Automatic conversions between different syntactic sorts

**Smart Constructors:** Auto-generated constructors with signature membership constraints

### Template Haskell Usage

The codebase extensively uses Template Haskell for deriving instances:
- `deriveAll` generates all standard instances for language fragments
- `createSortInclusionType` creates sort conversion types
- Smart constructors and pattern synonyms are auto-generated

### Adding new language support (based on tree-sitter)

1. In the tree-sitter grammar definition project root dir, you need to initialize project, with appropriate config file `tree-sitter.json` by running:
```bash
tree-sitter init
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cubix-framework/cubix](https://github.com/cubix-framework/cubix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
