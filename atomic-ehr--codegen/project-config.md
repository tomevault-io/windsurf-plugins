---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Commands

```bash
# Development
bun install                    # Install dependencies
bun test                      # Run all tests
bun test --watch              # Run tests in watch mode
bun test --coverage           # Run tests with coverage
bun run typecheck             # Type check the codebase
bun run lint                  # Lint and format code with Biome

# Building
bun run build                 # Build the project (creates dist/)
bun run src/cli/index.ts                   # Run CLI in development mode

# CLI Usage
bun run src/cli/index.ts typeschema generate hl7.fhir.r4.core@4.0.1 -o schemas.ndjson
bun run src/cli/index.ts generate typescript -i schemas.ndjson -o ./types
```

## Verification

After any code change, run at minimum:
```bash
bun run typecheck && bun run lint && bun test
```

For full verification including example generation and cross-project type checking:
```bash
make all
```
This runs: tests, lint with auto-fix, and all example generation pipelines (TypeScript R4, CCDA, SQL-on-FHIR, C#, Python, Mustache).

## Architecture Overview

This is a FHIR code generation toolkit with a **three-stage pipeline**:

### 1. Input Layer (`src/typeschema/`)
- **Parser** (`parser.ts`): Reads TypeSchema documents from files
- **Generator** (`generator.ts`): Converts FHIR packages to TypeSchema format
- **Core processors** (`core/`): Handle FHIR schema transformation
  - `transformer.ts`: Main FHIR-to-TypeSchema conversion
  - `field-builder.ts`: Builds TypeSchema fields from FHIR elements
  - `binding.ts`: Processes FHIR value bindings and enums
  - `nested-types.ts`: Handles nested type dependencies

### 2. High-Level API (`src/api/`)
- **APIBuilder** (`builder.ts`): Fluent interface for chaining operations
- **Generators** (`writer-generator/`): Language-specific code generators
  - `introspection.ts`: Generates introspection data like TypeSchema
  - `typescript.ts`: Generates TypeScript interfaces and types
  - `python.ts`: Generates Python/Pydantic models
  - `csharp.ts`: Generates C# classes

### 3. CLI Interface (`src/cli/`)
- **Commands** (`commands/`):
  - `typeschema`: Generate and validate TypeSchema from FHIR packages
  - `generate`: Generate code from TypeSchema (TypeScript, Python, C#)
- **Main entry** (`index.ts`): CLI setup with yargs

### Key Data Flow
```
FHIR Package → TypeSchema Generator → TypeSchema Format → Code Generators → Output Files
```

## Configuration

- **Main config**: `atomic-codegen.config.ts` (TypeScript configuration file)
- **Package config**: Uses `Config` type from `src/config.ts`
- **Default packages**: `hl7.fhir.r4.core@4.0.1`
- **Output dir**: `./generated` by default
- **Cache**: `.typeschema-cache/` for performance optimization

## Project Structure Patterns

- **TypeSchema types**: Defined in `src/typeschema/types.ts`
- **Tests**: Located in `test/unit/` with mirrors to `src/` structure
- **Generated code**: Output goes to `generated/` directory
- **Utilities**: Common functions in `src/utils.ts` and `src/typeschema/utils.ts`

## General Principles

- Bias toward action: start making changes directly. Do not write plan files, explore the entire codebase, or use Task sub-agents unless explicitly asked.
- Keep changes minimal and focused. Do not over-engineer (no extra abstractions, generics, or variants beyond what was requested). When in doubt, do the simplest thing that works.
- Only modify files and directories that were explicitly mentioned or are directly required by the change. Do not refactor surrounding code.
- When asked to review or explain code, explain first before proposing fixes. Do not jump to making changes unless explicitly asked to fix something.

## Commit Guidelines

- Split commits logically by concern. Always separate example/generated file updates from source code changes.
- Typical commit order: source changes → test changes → regenerated examples. Example updates should be the last commit in the branch.
- When making follow-up changes (fixes, refactors) to code already committed on the branch, create a new commit on top of the original commit with a prefix: `fix: ...`, `ref: ...`, etc. Do NOT amend or squash into the original commit — the user reviews changes step by step and will squash when finished.
- Never rewrite branch history (rebase, squash, amend) unless the user explicitly asks.

## Pull Request Style

- PR title must start with a module prefix: `TS:`, `PY:`, `C#:`, `TypeSchema:`, `CLI:`, `API:`, etc. to indicate which part of the codebase is affected. Use multiple prefixes (e.g. `TypeSchema/TS:`) when a change spans modules.
- PR body should be a bullet list summarizing changes — no test plan section.
  - Use two-level nesting to group related items when the list is long; keep it flat when short.
  - Use `##` section headers to group changes by concern when the PR spans multiple topics (e.g. renames, new features, config changes).
- Keep bullets concise and focused on what changed, not why.
- When a PR changes generated code or user-facing API, include before/after code examples.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atomic-ehr/codegen](https://github.com/atomic-ehr/codegen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
