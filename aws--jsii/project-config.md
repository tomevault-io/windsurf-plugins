---
trigger: always_on
description: AI agent contributor guide for the aws/jsii repository.
---

# AGENTS.md

AI agent contributor guide for the aws/jsii repository.

## Overview

- **jsii** enables TypeScript libraries to be consumed naturally in Python, Java, C#/.NET, and Go — users write idiomatic code in their language while jsii handles cross-language interop.
- **Key packages in this monorepo:**
  - `packages/@jsii/kernel` — Node.js process managing JS objects on behalf of host runtimes
  - `packages/@jsii/runtime` — Webpack-bundled runtime bootstrap
  - `packages/@jsii/spec` — Assembly schema definition (`.jsii` format)
  - `packages/@jsii/python-runtime` — Python host runtime
  - `packages/@jsii/java-runtime` — Java host runtime
  - `packages/@jsii/dotnet-runtime` — .NET host runtime
  - `packages/@jsii/go-runtime` — Go host runtime
  - `packages/jsii-pacmak` — Code generator producing language-specific packages from `.jsii` assemblies
  - `packages/jsii-reflect` — TypeScript reflection library for `.jsii` assemblies
  - `packages/jsii-diff` — API compatibility checker
  - `packages/codemaker` — Code generation utilities (indentation, file management)
  - `packages/jsii-calc`, `packages/@scope/*` — Test fixtures exercising the full type system
  - `tools/jsii-compliance` — Compliance suite and reporting tooling
  - `tools/jsii-build-tools` — Internal build utilities
- **Compiler and rosetta live in separate repositories:**
  - jsii compiler → https://github.com/aws/jsii-compiler
  - jsii-rosetta (transliteration) → https://github.com/aws/jsii-rosetta
- **Monorepo tooling:** Yarn 4 workspaces (`packageManager: yarn@4.13.0`) + Lerna for build orchestration
- **Design tenets:**
  - APIs must feel idiomatic in all target languages
  - Applications behave identically regardless of language (correctness over performance)
  - jsii does not attempt to support all TypeScript idioms — only those representable across all targets
  - Produced artifacts are compatible with each language's idiomatic package management tools

## Contributor Principles

1. Backwards compatibility of the jsii assembly format and kernel API is critical — never break existing consumers.
2. Cross-language correctness must be verified — a change affecting one language target must be validated against all targets.
3. Generated code must be idiomatic in each target language — follow each language's conventions and patterns.
4. The jsii type system restrictions exist to ensure cross-language representability — do not relax them without careful consideration and validation across all targets.
5. Snapshot tests must be updated when generated code changes — run `yarn test:update` to refresh snapshots.

## Quick Reference Commands

| Command | Description | Scope |
|---------|-------------|-------|
| `yarn install && yarn build` | Full bootstrap and build | Repo root |
| `cd packages/X && yarn build` | Build single package | Package dir |
| `yarn test` | Full test suite (includes compliance) | Repo root |
| `cd packages/X && yarn test` | Test single package | Package dir |
| `yarn test:update` | Update all snapshots | Repo root |
| `cd packages/X && yarn test:update` | Update package snapshots | Package dir |
| `yarn lint` | Lint all packages | Repo root |
| `yarn lint:fix` | Auto-fix lint issues | Repo root |
| `yarn compliance` | Generate compliance report | Repo root |
| `yarn upgrade:jsii` | Upgrade jsii/rosetta versions | Repo root |

**Notes:**

- Builds use `--concurrency=1` due to inter-package dependencies.
- Monorepo uses Yarn 4 workspaces + Lerna orchestration.
- For individual packages, use `cd packages/X && yarn <script>` (e.g., `cd packages/@jsii/kernel && yarn test`).

## Codebase Layout

### Packages

| Path | Purpose |
|------|---------|
| `packages/@jsii/kernel` | The JavaScript kernel — manages objects and executes code |
| `packages/@jsii/runtime` | IPC wrapper around kernel (webpack-bundled for distribution) |
| `packages/@jsii/spec` | Assembly schema definition and validation |
| `packages/@jsii/check-node` | Node.js version compatibility checks |
| `packages/@jsii/python-runtime` | Python host runtime library |
| `packages/@jsii/java-runtime` | Java host runtime library |
| `packages/@jsii/dotnet-runtime` | .NET host runtime library |
| `packages/@jsii/go-runtime` | Go host runtime library |
| `packages/jsii-pacmak` | Code generator (targets: Python, Java, .NET, Go, JS) |
| `packages/jsii-reflect` | Strongly-typed reflection library for `.jsii` assemblies |
| `packages/jsii-diff` | API backwards compatibility checker |
| `packages/jsii-config` | Interactive jsii configuration tool (experimental) |
| `packages/codemaker` | Code generation utilities (indentation, file management) |
| `packages/jsii-calc` | Test fixture library exercising the full type system |
| `packages/@scope/*` | Additional test fixture packages (base, lib, base-of-base) |
| `tools/jsii-compliance` | Compliance suite definition and report generation |
| `tools/jsii-build-tools` | Internal build utilities |

### Code Generation Targets

- Targets live in `packages/jsii-pacmak/lib/targets/`
- One file per language: `python.ts`, `java.ts`, `dotnet.ts`, `go.ts`, `js.ts`
- Complex targets have subdirectories: `dotnet/`, `go/`, `python/`

### Specifications & Docs

- Specification documents: `gh-pages/content/specification/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws/jsii](https://github.com/aws/jsii) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
