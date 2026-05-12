---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Semcode is a semantic code search tool written in Rust that indexes C/C++ codebases using machine learning embeddings. It consists of several binaries:
- `semcode-index`: Analyzes and indexes codebases using the CodeBERT model
- `semcode`: Interactive query tool for searching the indexed code
- `semcode-mcp`: Model Context Protocol server for Claude Desktop integration
- `semcode-lsp`: Language Server Protocol server for editor integration (see [docs/lsp-server.md](docs/lsp-server.md))

## Build and Development Commands

### Prerequisites
Install required system dependencies:
```bash
# Ubuntu/Debian
sudo apt-get install build-essential libclang-dev protobuf-compiler libprotobuf-dev

# Fedora
sudo dnf install gcc-c++ clang-devel protobuf-compiler protobuf-devel
```

**Note:** The C++ compiler (`g++`/`gcc-c++`) is required for building dependencies like `esaxx-rs`.

### Build Commands
```bash
# Build release binaries (recommended)
cargo build --release

# Use build script (creates symlinks in ./bin/)
./build.sh

# Build with test code samples
./build.sh --with-test
```

### Code Quality Commands

**MANDATORY before committing:**
```bash
# Format code (ALWAYS run this before committing)
cargo fmt

# Run clippy linter (treat warnings as errors)
cargo clippy --all-targets -- -D warnings
```

Clippy must pass with zero warnings before code can be committed. All clippy warnings should be fixed, not silenced with allow attributes unless there's a very good reason.

### Git Hooks

This repository uses shared git hooks to automatically enforce code formatting and linting. The hooks are tracked in the `hooks/` directory and shared with all developers.

**Setup for New Developers**

After cloning the repository, run the setup script to enable the hooks:

```bash
./setup-hooks.sh
```

Alternatively, you can manually configure the hooks:

```bash
git config core.hooksPath hooks
```

**Active Hooks**

- **pre-commit**: Runs comprehensive checks before each commit:
  - `cargo fmt --check` - Verifies code formatting
  - `cargo check --all-targets` - Verifies code compiles
  - `cargo test` - Runs all tests
- **pre-push**: Runs additional quality checks before each push:
  - `cargo fmt --check` - Verifies code formatting
  - `cargo clippy --all-targets -- -D warnings` - Checks for code quality issues
  - `cargo test` - Runs all tests

These hooks ensure that improperly formatted, non-compiling, or failing code cannot be committed or pushed to the repository.

**If a hook fails:**

1. For formatting issues: Run `cargo fmt` to format your code
2. For compilation errors: Run `cargo check --all-targets` to see the errors and fix them
3. For clippy issues: Fix the warnings/errors reported by clippy
4. For test failures: Fix the failing tests
5. Stage your changes with `git add` (if needed)
6. Try your commit or push again

**Hook Location**

The git hooks are stored in the `hooks/` directory (tracked by git) and are shared across all developers. This ensures consistent code quality enforcement for everyone working on the project.

### Database Location

Semcode uses the following search order to locate the `.semcode.db` database directory:

**For semcode-index:**
1. **-d flag**: If provided, use the specified path (direct database path or parent directory containing `.semcode.db`)
2. **SEMCODE_DB environment variable**: Same path semantics as `-d`
3. **Source directory**: Look for `.semcode.db` in the source directory specified by `-s`
4. **Current directory**: Fall back to `./.semcode.db` in the current working directory

**For semcode (query tool), semcode-mcp, and semcode-lsp:**
1. **-d flag / configuration**: If provided, use the specified path (direct database path or parent directory containing `.semcode.db`)
2. **SEMCODE_DB environment variable**: Same path semantics as `-d`
3. **Workspace/Current directory**: Use `./.semcode.db` in the workspace or current working directory

The `-d` flag can specify either:
- A direct path to the database directory (e.g., `./my-custom.db`)
- A parent directory containing `.semcode.db` (e.g., `-d /path/to/project` will use `/path/to/project/.semcode.db`)

### Running the Tools

**Typical workflow:**
```bash
# Index a codebase - creates /path/to/code/.semcode.db
./bin/semcode-index --source /path/to/code

# Query from within the indexed directory
cd /path/to/code
semcode  # Automatically finds ./.semcode.db

# Or query from elsewhere
semcode --database /path/to/code  # Uses /path/to/code/.semcode.db
```

**Other indexing options:**
```bash
# Basic analysis only (no vectors)
./bin/semcode-index --source /path/to/code

# Index files modified in git commit range
./bin/semcode-index --source /path/to/code --git HEAD~10..HEAD

# Custom database location (overrides search order)
./bin/semcode-index --source /path/to/code --database ./custom.db
```

## Architecture

### Git-Aware Operations (IMPORTANT)

**All features that query the database MUST use git-aware lookup functions.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [facebookexperimental/semcode](https://github.com/facebookexperimental/semcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
