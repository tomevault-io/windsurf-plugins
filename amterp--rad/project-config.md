---
trigger: always_on
description: - Where the syntax of the Rad language itself becomes relevant, see SYNTAX.md for reference.
---

- Where the syntax of the Rad language itself becomes relevant, see SYNTAX.md for reference.

- You have the following subagents to request input from:
  - Code Reviewer (for when you make large changes)
  - Rad Docs Maintainer (for when you make user-facing changes)

- You have the following useful commands available to you:
  - `make format` 
  - `make build`: builds the project into a local test binary `./bin/radd`
  - `make test`
  - `./dev --validate`: Runs `go mod tidy`, formats, builds, and runs tests.

- Please do not leave task-specific messages to the user via comments in the code base when making changes.

- Never commit `replace` directives in `go.mod`. These are used locally during development to point at local
  copies of dependencies, but must be removed before committing.

---

## Pre-Commit Checklist

A Claude Code hook will remind you of this checklist when you commit. Review every item; skip categories that
don't apply to your change.

### Always
- Run `./dev --validate` (formats, builds, tests). All tests pass.
- Commit messages follow conventional prefixes (`feat:`, `fix:`, `refactor:`, `docs:`, `test:`).
- Commit messages explain **why**, not just what. See `CONTRIBUTING.md` for full conventions.

### When Adding or Modifying a Built-in Function
- Function documented in `docs/funcs/<name>.md` (source of truth - see `docs/funcs/README.md` for the required format). The signature line in `## Signature` IS the type-checker's signature; there is no parallel definition in `rts/signatures.go` anymore.
- Snapshot tests added/updated in `core/testing/snapshots/functions/<name>.snap`.
- Run `make generate`. This regenerates `rts/signatures_gen.go`, mirrors `docs/funcs/` into `rts/embedded_funcs/`, regenerates the public reference at `docs-web/docs/reference/functions.md`, and refreshes `rts/embedded/functions.txt`. Commit the regenerated artifacts.

### When Changing Language Syntax or Semantics
- `SYNTAX.md` updated to reflect the change.
- Snapshot tests added in the appropriate `core/testing/snapshots/` subdirectory.
- If AST nodes were added/changed, parser snapshot tests in `rts/test/st_snapshots/` updated.
- Guide docs updated if the feature has a section in `docs-web/docs/guide/`.

### When Introducing a Breaking Change
- Commit message uses `feat!:` or `fix!:` prefix.
- Migration guide entry added to the current version's `docs-web/docs/migrations/` file.
- Migration diagnostic added (see [Breaking Changes & Migration Diagnostics](#breaking-changes--migration-diagnostics)).

### When Adding or Modifying Error Codes
- Error doc file created/updated in `core/error_docs/<code>.md`.
- Error code defined in `rts/rl/errors.go` if new.

### When Touching Platform-Specific Behavior
- Logic centralized in `core/common/platform.go`, not scattered via `runtime.GOOS` checks.
- Paths returned to user code are normalized via `NormalizePath()`.
- Platform-specific tests in `core/testing/platform_test.go` if applicable.

---

# Rad Language - LLM Quick Reference

**Rad is a modern CLI scripting language designed to replace Bash for most scripting needs.**

## Project Overview

Rad (🤙 Rad) is a lightweight CLI scripting language that makes shell scripting easier, more readable, and more
maintainable than Bash. It combines familiar Python-like syntax with powerful scripting-specific features.

### Key Features

- **Declarative argument parsing** with automatic help generation
- **Built-in JSON processing** with path expressions
- **HTTP request syntax** (`rad url`) for API interactions
- **Table formatting** and data display
- **String interpolation** with `{variable}` syntax
- **Shell command integration** while avoiding Bash pitfalls
- **Type system** with runtime type checking
- **Interactive prompts** via `pick()` function

## Project Structure

```
├── main.go                    # Entry point - creates RadRunner
├── go.mod                     # Go module definition
├── Makefile                   # Build system (generate, format, build, test)
├── README.md                  # User documentation
├── core/                      # Interpreter (evaluates AST, no tree-sitter)
│   ├── runner.go              # Main runner logic
│   ├── interpreter.go         # AST evaluation via Go type switch
│   ├── funcs.go              # Built-in functions
│   ├── rad_block.go          # Rad block syntax (HTTP requests)
│   ├── args.go               # Argument parsing
│   ├── json_*.go             # JSON processing algorithms
│   ├── type_*.go             # Type system implementation
│   └── testing/              # Comprehensive test suite
├── rts/                      # Parsing, conversion, and static analysis
│   ├── parse.go              # Tree-sitter parser wrapper
│   ├── converter.go          # CST-to-AST single-pass converter
│   ├── nodes.go              # CST node types and traversal
│   ├── signatures.go         # Built-in function signatures
│   ├── check/                # Static checker (AST-based, CST fallback)
│   └── rl/                   # AST node types, spans, typing, node kinds
├── radls/                    # Language Server Protocol implementation
├── vsc-extension/            # VS Code extension
├── docs-web/                 # Documentation website (MkDocs)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amterp/rad](https://github.com/amterp/rad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
