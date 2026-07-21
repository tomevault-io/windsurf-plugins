---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## Project Overview

Cog is a tool that packages machine learning models in production-ready containers.

It consists of:

- **Cog CLI** (`cmd/cog/`) - Command-line interface for building, running, and deploying models, written in Go
- **Python SDK** (`python/cog/`) - Python library for defining model predictors and training in Python
- **Coglet** (`crates/`) - Rust-based prediction server that runs inside containers, with Python bindings via PyO3

Documentation for the CLI and SDK is available by reading ./docs/llms.txt.

## Development Commands

Development tasks are managed with [mise](https://mise.jdx.dev/). Run `mise tasks` to see all available tasks.

### Quick Reference

| Task                          | Description                                                  |
| ----------------------------- | ------------------------------------------------------------ |
| `mise run fmt`                | Check formatting (all languages)                             |
| `mise run fmt:fix`            | Fix formatting (all languages)                               |
| `mise run lint`               | Run linters (all languages)                                  |
| `mise run lint:fix`           | Fix lint issues (all languages)                              |
| `mise run test:go`            | Run Go tests                                                 |
| `mise run test:rust`          | Run Rust tests                                               |
| `mise run test:python`        | Run Python tests                                             |
| `mise run test:integration`   | Run integration tests                                        |
| `mise run build:cog`          | Build cog CLI binary                                         |
| `mise run build:coglet`       | Build coglet wheel (dev)                                     |
| `mise run build:sdk`          | Build SDK wheel                                              |
| `mise run install`            | Build and symlink cog to /usr/local/bin                      |
| `mise run docs:llm`           | **IMPORTANT:** Regenerate `docs/llms.txt` after editing docs |
| `mise run docs:cli`           | Generate CLI reference docs from Go source code              |
| `mise run version`            | Show current version from VERSION.txt                        |
| `mise run version:bump <ver>` | Bump version everywhere and commit                           |

### Task Naming Convention

Tasks follow a consistent naming pattern:

- **Language-based tasks** for fmt/lint/test/typecheck: `task:go`, `task:rust`, `task:python`
- **Component-based tasks** for build: `build:cog`, `build:coglet`, `build:sdk`
- **Check vs Fix**: `fmt` and `lint` default to check mode (non-destructive); use `:fix` suffix to auto-fix

### All Tasks by Category

**Format:**

- `mise run fmt` / `mise run fmt:check` - Check all (alias)
- `mise run fmt:fix` - Fix all
- `mise run fmt:go` / `mise run fmt:rust` / `mise run fmt:python` - Per-language

**Lint:**

- `mise run lint` / `mise run lint:check` - Check all (alias)
- `mise run lint:fix` - Fix all
- `mise run lint:go` / `mise run lint:rust` / `mise run lint:python` - Per-language
- `mise run lint:rust:deny` - Check Rust licenses/advisories

**Test:**

- `mise run test:go` - Go unit tests
- `mise run test:rust` - Rust unit tests
- `mise run test:python` - Python unit tests (via tox)
- `mise run test:coglet:python` - Coglet Python binding tests
- `mise run test:integration` - Integration tests

**Build:**

- `mise run build:cog` - Build cog CLI (development)
- `mise run build:cog:release` - Build cog CLI (release)
- `mise run build:coglet` - Build coglet wheel (dev install)
- `mise run build:coglet:wheel` - Build coglet wheel (native platform)
- `mise run build:coglet:wheel:linux-x64` - Build for Linux x86_64
- `mise run build:coglet:wheel:linux-arm64` - Build for Linux ARM64
- `mise run build:sdk` - Build SDK wheel

**Install:**

- `mise run install` - Symlink cog CLI to `/usr/local/bin` (requires `build:cog` first)
- `PREFIX=/custom/path mise run install` - Symlink to custom location

**Version:**

- `mise run version` - Show current version from VERSION.txt
- `mise run version:bump <ver>` - Bump version everywhere and commit
- `mise run version:check` - Verify VERSION.txt matches Cargo.toml

**Other:**

- `mise run typecheck` - Type check all languages
- `mise run generate` - Run code generation
- `mise run clean` - Clean all build artifacts
- `mise run docs` - Build documentation
- `mise run docs:serve` - Serve docs locally

## Code Style Guidelines

### Go

- **Imports**: Organize in three groups separated by blank lines: (1) Standard library, (2) Third-party packages, (3) Internal packages (`github.com/replicate/cog/pkg/...`)
- **Formatting**: Use `mise run fmt:go:fix`
- **Linting**: Must pass golangci-lint with: errcheck, gocritic, gosec, govet, ineffassign, misspell, revive, staticcheck, unused
- **Error Handling**: Return errors as values; use `pkg/errors.CodedError` for user-facing errors with error codes
- **Naming**: CamelCase for exported, camelCase for unexported

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [replicate/cog](https://github.com/replicate/cog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
