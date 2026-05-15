---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## CRITICAL PRIORITY INSTRUCTIONS

**MOST IMPORTANT**: When evaluating Gleam script execution via `deno task cli`, NEVER use simulation. Always base script evaluation on the actual Deno execution results. The runner executes real HTTP requests and should return real data, not simulated output.

- Execute scripts using `deno task cli <script.gleam>`
- Evaluate success/failure based on actual console output
- Debug issues using real execution results
- No simulation or fake data generation allowed

## Project Overview

This is "subaru" - a Gleam WASM runner that allows executing Gleam code dynamically using WebAssembly. The project uses:

- **Gleam**: Main programming language (functional language that compiles to Erlang/JavaScript)
- **Deno**: TypeScript/JavaScript runtime for the WASM runner
- **WASM**: WebAssembly version of Gleam compiler for dynamic compilation
- **Nix Flakes**: Development environment management with devenv
- **Pre-commit hooks**: Security scanning with git-secrets and ripsecrets

## Development Commands

### Setup

```bash
deno task setup     # Download Gleam WASM compiler
```

### Gleam Operations

```bash
gleam run      # Run the main Gleam application
gleam test     # Run Gleam tests using gleeunit
```

### WASM Runner Operations

```bash
# CLI usage
deno task cli --help                    # Show CLI help
deno task cli example.gleam             # Execute Gleam file directly (preferred)
deno task cli --file example.gleam     # Execute Gleam code from file (alternative)  
deno task cli --code "gleam_code_here"  # Execute Gleam code directly
deno task cli --url https://example.com/script.gleam  # Execute remote script

# Debug control (silent is default)
deno task cli --debug --code "..."      # Enable debug output
deno task cli --log-level error --code "..."   # Show compilation errors/warnings

# Configuration
deno task init-config                   # Create example config file
deno task cli --config my-config.json script.gleam  # Use custom config with direct file

# Examples and testing
deno task example                       # Run usage examples
deno task example:debug                 # Run debug mode examples
deno task example:preload               # Run preload scripts example
deno task test                          # Run Deno tests
```

### Development Environment

```bash
deno task dev        # Setup and run development environment
# OR use Nix/direnv for reproducible environment:
nix develop          # Enter the development shell (if using Nix)
direnv allow         # Auto-load development environment (if direnv is configured)
```

### Code Quality

```bash
deno task fmt            # Format TypeScript code
deno task lint           # Lint TypeScript code
deno task check          # Type check TypeScript code
deno task test           # Run Deno tests
deno task build-gleam    # Build Gleam project
deno task run-gleam      # Run Gleam project
deno task clean          # Clean generated files
nix run .#treefmt        # Format Nix code
git secrets --scan       # Scan for secrets (pre-commit hook)
```

**IMPORTANT**: Always run `deno fmt` and `deno test` during development before committing changes. These commands should be used frequently to catch formatting issues and test failures early.

## Project Structure

### Gleam Files

- `src/subaru.gleam`: Main Gleam application entry point
- `test/subaru_test.gleam`: Gleam test suite using gleeunit

### TypeScript/WASM Runner

- `src/gleam_runner.ts`: Core WASM runner implementation
- `src/subaru_runner.ts`: High-level API for Gleam code execution
- `src/cli.ts`: Command-line interface for the runner
- `src/hex/`: Hex.pm package integration
  - `hex_client.ts`: Hex.pm API client
  - `tarball_extractor.ts`: Hex tarball extraction
  - `package_cache.ts`: Package caching
- `src/stdlib/`: Standard library loading
  - `builtin_packages.ts`: Builtin package definitions
  - `stdlib_loader.ts`: Library loading orchestration
- `test/subaru_runner_test.ts`: Deno tests for WASM functionality
- `test/hex/`: Tests for Hex.pm integration
- `test/stdlib/`: Tests for stdlib loading
- `examples/simple_usage.ts`: Usage examples

### Configuration & Scripts

- `gleam.toml`: Gleam project configuration and dependencies
- `deno.json`: Deno configuration and development task definitions
- `flake.nix`: Nix development environment with custom Gleam build
- `src/setup.ts`: WASM compiler setup script

## Architecture Notes

- **Dual Runtime**: Gleam for static compilation, Deno for dynamic WASM execution
- **WASM Integration**: Uses Gleam's WebAssembly compiler for dynamic code compilation
- **Worker-based Execution**: Isolates compiled JavaScript execution in Web Workers
- **CLI Interface**: Provides easy command-line access to WASM functionality
- **Testing Strategy**: Gleam tests for static code, Deno tests for WASM functionality
- **Custom Gleam Build**: The flake.nix builds Gleam v1.9.1 from source using Rust nightly
- **Security**: Pre-commit hooks scan for secrets using git-secrets and ripsecrets

## Key Features


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Comamoca/subaru](https://github.com/Comamoca/subaru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
