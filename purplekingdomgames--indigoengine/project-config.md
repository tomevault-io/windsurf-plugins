---
trigger: always_on
description: This is a **Mill-based monorepo** containing Scala libraries and frameworks by Purple Kingdom Games:
---

# Copilot Instructions for Indigo Engine Monorepo

## Repository Overview

This is a **Mill-based monorepo** containing Scala libraries and frameworks by Purple Kingdom Games:

- **Indigo**: Functional game engine built on Scala.js + WebGL (300+ files)
- **Tyrian**: Elm-inspired Scala UI library for Scala 3 (100+ files)
- **Ultraviolet**: Scala 3 to GLSL transpiler (50+ files)
- **Roguelike Starter Kit**: ASCII art roguelike rendering for Indigo (50+ files)

**Languages**: Scala 3 (version defined in `build.mill`), targeting both JVM and JavaScript via Scala.js  
**Build Tool**: Mill (version defined in `.mill-version`, launcher script: `./mill` on Unix, `.\mill.bat` on Windows)  
**Testing**: MUnit framework  
**Java Version**: Java 11+ required (CI uses Java 11; Java 17 also works)

## Critical Build Information

### ⚠️ Target Specific Modules - Avoid Wasting Time

**CRITICAL:** This is a large monorepo with many modules. **Always compile/build only the specific modules you're working on** rather than using general commands like `__.compile`, `__.test`, etc.

```bash
# ✅ CORRECT - Target specific modules
./mill indigo-core.js.compile
./mill ultraviolet.jvm.compile
./mill tyrian.js.compile
./mill indigo-core.js.test

# ❌ AVOID - Compiles/tests ALL modules (extremely slow and wasteful)
./mill __.compile
./mill __.test
```

**Module targeting examples:**
- Working on Ultraviolet? Use `./mill ultraviolet.js.compile` and `./mill ultraviolet.jvm.compile`
- Working on Indigo core? Use `./mill indigo-core.js.compile`
- Working on Tyrian? Use `./mill tyrian.js.compile`
- Need to check dependencies? Target the specific module plus its dependencies (e.g., `./mill indigo.js.compile` for Indigo and its deps)

**When to use general builds:**
- **Only** for pre-commit validation or when impact is uncertain across the entire codebase
- The `build.sh` and `ci.sh` scripts are intentionally comprehensive for full validation
- For iterative development, **always target specific modules**

### Resource-Intensive Operations

**⚠️ ALWAYS limit concurrency for Scala.js linking operations** - full parallelism exhausts system resources:

```bash
# CORRECT - limit concurrency to 2 jobs
./mill -j2 __.fastLinkJS
./mill -j2 __.fastLinkJSTest
./mill -j2 __.fix

# INCORRECT - will likely hang or fail
./mill __.fastLinkJS
```

For other operations, full parallelism is fine:
```bash
./mill __.compile     # Full parallelism OK
./mill __.test        # Full parallelism OK
```

### Essential Build Commands

**Bootstrap**: No special bootstrap needed. Mill launcher downloads Mill automatically on first run.

**Full Build Sequence** (validated by `build.sh`):
```bash
# 1. Compile all modules (2-5 minutes)
./mill --no-server __.compile

# 2. Format code
./mill --no-server __.reformat

# 3. Run scalafix (limited concurrency - 3-5 minutes)
./mill --no-server -j2 __.fix

# 4. Link Scala.js (limited concurrency - 5-10 minutes)
./mill --no-server -j2 __.fastLinkJS
./mill --no-server -j2 __.fastLinkJSTest

# 5. Run tests (5-15 minutes)
./mill --no-server __.test

# 6. Publish locally
./mill --no-server __.publishLocal
```

**Quick validation script**: `bash build.sh` runs the full sequence above.

**CI Validation** (what GitHub Actions runs):
```bash
bash ci.sh
```

This runs the same steps as `build.sh` but with additional flags (`clean`, `--disable-ticker`, `checkFormat` instead of `reformat`, `fix --check` instead of `fix`).

### Individual Commands

**Compile**: `./mill __.compile` (compiles all modules)  
**Test All**: `./mill __.test` (runs all tests)  
**Test Single Module**: `./mill indigo.js.test` or `./mill ultraviolet.jvm.test`  
**Test Specific Class**: `./mill indigo-core.js.test.testOnly indigo.core.SomeTests`  
**Format Code**: `./mill __.reformat` (applies formatting)  
**Check Format**: `./mill __.checkFormat` (validates formatting without changes)  
**Run Scalafix**: `./mill -j2 __.fix` (applies scalafix rules with limited concurrency)  
**Check Scalafix**: `./mill -j2 __.fix --check` (validates scalafix without changes)

### --no-server Flag

The `--no-server` flag is recommended for stability. If you encounter `NoClassDefFoundError` in Scala.js testing classes (e.g., `Serializer$SelectorSerializer$`), this is a Mill server communication issue. Running with `--no-server` avoids it.

### Windows Considerations

On Windows, use `.\mill.bat` instead of `./mill` for all commands. The PowerShell build script `build.ps1` uses `-j1` (single-threaded) concurrency for all operations for maximum stability.

## Code Style and Quality

### Formatting and Linting

- **Scalafmt**: Configured in `.scalafmt.conf`
  - Dialect: `scala3`
  - Max column: 120
  - Always run before committing: `./mill __.reformat`

- **Scalafix**: Configured in `.scalafix.conf`
  - Rules: `DisableSyntax`, `OrganizeImports`
  - **Enforces functional style**: No vars, throws, nulls, returns, while loops, XML, default args
  - Always run before committing: `./mill -j2 __.fix`

### Strict Compilation

- **Scala 3** with strict equality enabled
- Compiler treats warnings as errors (`-Werror`)
- Uses typelevel scalacoptions for consistent, strict compilation flags

## Project Architecture

### Module Structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PurpleKingdomGames/indigoengine](https://github.com/PurpleKingdomGames/indigoengine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
