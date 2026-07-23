---
trigger: always_on
description: - [Git workflow](#git-workflow)
---

# Deno Development Guide

## Table of Contents

- [Git workflow](#git-workflow)
- [High Level Overview](#high-level-overview)
- [Quick Start](#quick-start)
- [Commands](#commands)
- [Testing](#testing)
- [Development Workflows](#development-workflows)
- [Debugging](#debugging)
- [Codebase Navigation](#codebase-navigation)
- [Troubleshooting](#troubleshooting)

## Git workflow

Deno uses a GH based standard git workflow. The main branch is `main`. All
development happens in feature branches, which are then merged into `main` via
pull requests.

When the feature is finished and ready to for review, follow these steps:

- Create a new git branch, if you haven't already, with a descriptive name
  (e.g., `feature/new-cli-command` or `fix/bug-in-worker-threads`).
- Commit your changes with clear and descriptive commit messages.
- Push your branch to the remote repository.
- Open a pull request (PR) against the `main` branch on GitHub.
- Before committing, make sure `tools/format.js` is run to format your code
- Before committing, if only non-Rust was changed, make sure to run
  `tools/lint.js --js` and fix any lint errors before committing
- If you changed Rust code, make sure to run `tools/lint.js` and fix any lint
  errors before committing
- In the PR description, provide a clear summary of the changes you made, why
  they were necessary, and any relevant context or links to related issues.
- When pushing updates to the PR, make sure to never force push. Create as many
  commits as you need, all of them get squashed when the PR is merged, so there
  is no need to rewrite history. This also allows reviewers to see the
  incremental changes you made in response to feedback.
- Keep your changes minimal, don't do drive-by changes in a PR. If you need to
  make a change that is not directly related to the PR, create a separate PR for
  it. This keeps the review process focused and efficient.

## High Level Overview

The user visible interface and high level integration is in the `deno` crate
(located in `./cli`).

This includes flag parsing, subcommands, package management tooling, etc. Flag
parsing is in `cli/args/flags.rs`. Tools are in `cli/tools/<tool>`.

The `deno_runtime` crate (`./runtime`) assembles the JavaScript runtime,
including all "extensions" (native functionality exposed to JavaScript). The
extensions themselves are in the `ext/` directory, and provide system access to
JavaScript – for instance filesystem operations and networking.

### Key Directories

- `cli/` - User-facing CLI implementation, subcommands, and tools
- `runtime/` - JavaScript runtime assembly and integration
- `ext/` - Extensions providing native functionality to JS (fs, net, etc.)
- `tests/specs/` - Integration tests (spec tests)
- `tests/unit/` - Unit tests
- `tests/testdata/` - Test fixtures and data files

## Quick Start

Before building, install the required prerequisites (Rust, native compilers,
cmake, protobuf, etc.) and clone with `--recurse-submodules` as described in
[`.github/CONTRIBUTING.md`](.github/CONTRIBUTING.md#building-from-source).

### Building Deno

To compile after making changes:

```bash
cargo build
```

For faster iteration during development (less optimization):

```bash
cargo build --bin deno
```

Execute your development build:

```bash
./target/debug/deno eval 'console.log("Hello from dev build")'
```

### Running with your changes

```bash
# Run a local file
./target/debug/deno run path/to/file.ts

# Run with permissions
./target/debug/deno run --allow-net --allow-read script.ts

# Run the REPL
./target/debug/deno
```

## Commands

### Compilation and Checks

```bash
# Check for compilation errors (fast, no binary output)
cargo check

# Check specific package
cargo check -p deno_runtime

# Build release version (slow, optimized)
cargo build --release
```

### Code Quality

```bash
# Lint the code
./tools/lint.js

# Format the code
./tools/format.js

# Both lint and format
./tools/format.js && ./tools/lint.js
```

## Testing

### Running Tests

```bash
# Run all tests (this takes a while)
cargo test

# Filter tests by name
cargo test <nameOfTest>

# Run tests in a specific package
cargo test -p deno_core

# Run just the CLI integration tests
cargo test --bin deno

# Run spec tests only
cargo test specs

# Run a specific spec test
cargo test spec::test_name
```

### Unit Tests (`tests/unit/`)

JavaScript/TypeScript unit tests live in `tests/unit/` as `*_test.ts` files. Run
them via `cargo test`:

```bash
# Run all unit tests in a specific file
cargo test unit::webcrypto_test

# Run all unit tests
cargo test unit::

# Run Node.js compatibility unit tests (tests/unit_node/)
cargo test unit_node::crypto_test

# Run all Node.js compat unit tests
cargo test unit_node::
```

Do NOT run these directly with `./target/debug/deno test` — they depend on the
cargo test harness for correct setup.

### Test Organization

- **Spec tests** (`tests/specs/`) - Main integration tests, CLI command
  execution and output validation
- **Unit tests** (`tests/unit/`) - JavaScript/TypeScript unit tests for runtime
  APIs
- **Integration tests** (`cli/tests/`) - Additional integration tests
- **WPT** (`tests/wpt/`) - Web Platform Tests for web standards compliance

## "spec" tests


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [denoland/deno](https://github.com/denoland/deno) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
