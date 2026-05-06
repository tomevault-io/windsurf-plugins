---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# @printers/printers - AI Agent Guidance

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Project Overview

This is a **cross-runtime printer library** for JavaScript with **Node.js as the primary runtime**,
also supporting **Deno** and **Bun** with a unified API. All runtimes use
N-API native bindings with the same interface.

## Quick Start Commands

### Primary Entrypoint

**⭐ Always use `index.ts` as the primary entrypoint** - it auto-detects the
runtime and loads the appropriate implementation:

```bash
# Universal entrypoint (RECOMMENDED - works in all runtimes)
npx tsx src/index.ts                            # Node.js with N-API (PRIMARY)
bun src/index.ts                                # Bun with N-API
deno run --allow-env --allow-ffi src/index.ts  # Deno with N-API (--allow-ffi required)
```

### CI and Status Checks

The CI system provides comprehensive testing with:

- **Cross-runtime compatibility tests** across Deno, Bun, and Node.js
- **Rust library unit tests** with simulation mode
- **Automated PR status checks** with detailed test result reporting
- **LCOV coverage analysis** with actual percentage calculations
- **JUnit XML test reports** for all runtimes

### Building and Testing

```bash
# Build N-API module and compile TypeScript (recommended)
task build

# Build N-API module only
task build:napi          # Build N-API module for all runtimes

# Test all runtimes with comprehensive reporting (generates JUnit XML + LCOV coverage)
task test                # All runtimes via test-runtimes.js
task test -- rust        # Only Rust tests via test-runtimes.js
task test -- deno node bun  # Only JavaScript runtimes via test-runtimes.js

# Test individual runtimes directly (verbose output, no reporting)
task test:direct         # All runtimes with direct commands
task test:direct:rust    # cargo test
task test:direct:deno    # deno test
task test:direct:node    # Node.js test runner
task test:direct:bun     # bun test
```

### Development Workflow

```bash
# Run CI locally with nektos/act
task ci:local

# Format code
task fmt

# Lint code
task lint

# Type check all entry points
task check:all
```

### Version Management

```bash
task bump:patch    # 0.1.4 -> 0.1.5
task bump:minor    # 0.1.4 -> 0.2.0
task bump:major    # 0.1.4 -> 1.0.0
```

## Architecture Summary

### Primary Entry Point

- **`src/index.ts`**: ⭐ **PRIMARY UNIVERSAL ENTRY POINT** - auto-detects runtime
  and loads N-API implementation for all runtimes. Always use this for consistent behavior.

### Backend

- **`lib/core.rs`**: Shared business logic for all runtimes
- **`lib/node.rs`**: N-API bindings for all JavaScript runtimes
- **`lib/napi.rs`**: N-API module definitions

### Testing

- **`tests/shared.test.ts`**: Cross-runtime test suite using src/index.ts
- **`tests/node-test-runner.mjs`**: Custom Node.js test runner with TypeScript
  support and c8 coverage generation

### Automation

**Node.js Scripts** (ESM JavaScript):

- **`scripts/build-all.js`**: Cross-runtime build orchestration
- **`scripts/test-runtimes.js`**: Comprehensive test runner with coverage reports
- **`scripts/run-ci-local.js`**: Local CI simulation
- **`scripts/bump-version.js`**: Version management
- **`scripts/build-napi.js`**: N-API module building (requires Node.js subprocess environment)
- **`scripts/remove-env-check.js`**: Post-build N-API processing
- **`scripts/compile.js`**: TypeScript to JavaScript compilation for npm publishing
- **`scripts/utils.js`**: Shared utility functions for build scripts

All utility scripts are Node.js JavaScript files using ESM syntax.

## Safety Reminders

⚠️ **This library sends real print jobs to physical printers!**

- Always use `PRINTERS_JS_SIMULATE=true` for safe testing
- Default tests use simulation mode automatically
- Scripts like `test-runtimes.js` automatically set simulation mode
- Only disable simulation mode when intentionally testing real printing
- The devcontainer sets `PRINTERS_JS_SIMULATE=true` by default

## Code Quality Requirements

**⚠️ IMPORTANT: Always use the taskfile commands for formatting and linting!**

ALWAYS run these after changes through the taskfile:

- `task fmt` - Format all code with **Prettier** (TypeScript/JavaScript) and cargo fmt (Rust)
- `task lint` - Lint all code with **ESLint** (TypeScript/JavaScript) and cargo clippy (Rust)
- `task check:all` - Type check all entry points

**DO NOT run formatters/linters directly** - always use the taskfile to ensure consistency:

- ❌ Never run `prettier` directly
- ❌ Never run `eslint` directly
- ❌ Never run `deno fmt` or `deno lint`
- ✅ Always use `task fmt` for formatting
- ✅ Always use `task lint` for linting

**Formatting and Linting Tools:**

- **Prettier** - Primary formatter for all TypeScript/JavaScript code
- **ESLint** - Primary linter for all TypeScript/JavaScript code
- **cargo fmt** - Formatter for Rust code
- **cargo clippy** - Linter for Rust code

## File Organization

- **`src/`**: TypeScript implementation files with modular architecture
- **`lib/`**: Rust source code with modular architecture
- **`tests/`**: Test files organized by runtime and purpose
- **`scripts/`**: Node.js JavaScript build automation and utility scripts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [esimkowitz/printers-js](https://github.com/esimkowitz/printers-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
