---
trigger: always_on
description: This document provides guidance for AI coding assistants (like GitHub Copilot, Cursor, Claude, etc.) when working with the oxc-resolver codebase.
---

# AI Agent Guidelines for oxc-resolver

This document provides guidance for AI coding assistants (like GitHub Copilot, Cursor, Claude, etc.) when working with the oxc-resolver codebase.

## Project Overview

oxc-resolver is a Rust port of webpack's enhanced-resolve, providing ESM and CommonJS module resolution. It offers both a Rust crate and Node.js bindings via NAPI.

### Key Technologies

- **Rust**: Core implementation using Rust 2024 edition (MSRV: 1.95.0)
- **NAPI**: Node.js bindings for JavaScript/TypeScript usage
- **WebAssembly**: Browser support
- **GitHub Actions**: CI/CD workflows

## Architecture

```
oxc-resolver/
├── src/                 # Core Rust implementation
│   └── tests/           # Unit tests (one file per feature area)
├── tests/               # Integration tests
├── fixtures/            # Test fixtures (real files on disk, not generated)
├── napi/               # Node.js NAPI bindings
├── examples/           # Usage examples
├── benches/            # Performance benchmarks
└── .github/            # GitHub workflows and configs
```

## Development Workflow

`just init` has already been run, all tools (`typos-cli`, `cargo-shear`) are already installed, do not run `just init`.

Rust and `cargo` components `clippy`, `rust-docs` and `rustfmt` have already been installed, do not install them.

Always run `just ready` as the last step after code has been committed to the repository.

### Common Tasks

```bash
just ready     # Run all checks (format, lint, test, build)
just test      # Run all tests (Rust + Node.js)
just check     # Cargo check with all features
just lint      # Run clippy with strict settings
just fmt       # Format code (cargo fmt + vp fmt)
```

## Code Conventions

### Rust

- Use Rust 2024 edition features
- Follow standard Rust formatting (`cargo fmt`)
- All clippy warnings must be addressed (`cargo clippy -- --deny warnings`)
- Use `tracing` for logging/instrumentation
- Implement `FileSystem` trait for custom file systems

### Node.js/TypeScript

- Use TypeScript for type definitions (`index.d.ts`)
- Follow existing API patterns in NAPI bindings
- Use vitest for testing
- Support both ESM and CommonJS usage

### Documentation

- Use rustdoc for Rust APIs
- Maintain TypeScript definitions for Node.js API
- Update README.md for significant changes
- Add examples for new features

## Key APIs

### Rust

```rust
use oxc_resolver::{ResolveOptions, Resolver};

let options = ResolveOptions::default();
let resolver = Resolver::new(options);
let resolution = resolver.resolve("/path/to/project", "./module");
```

### Node.js

```javascript
import resolve, { ResolverFactory } from "oxc-resolver";

// Simple resolve
const result = resolve.sync(process.cwd(), "./module");

// Advanced usage
const resolver = new ResolverFactory({
  conditionNames: ["node", "import"],
  extensions: [".js", ".ts", ".json"],
});
```

## Testing Strategy

### Test Categories

1. **Enhanced-resolve compatibility**: Tests ported from webpack/enhanced-resolve
2. **TypeScript support**: tsconfig-paths functionality
3. **Yarn Plug'n'Play** resolution
4. **Node.js compatibility**: ESM/CJS resolution behavior
5. **Performance**: Benchmarks against enhanced-resolve

### Adding Tests

Tests must use **fixture directories** with real files on disk. Do not dynamically create files, directories, or temp folders in tests — always add fixture files and commit them to the repository.

#### Where to put test code

- **Unit tests** (`src/tests/`): Test individual resolution features (aliases, extensions, exports, etc.). Each file maps to a feature area. Use `super::fixture_root()` to access `fixtures/`.
- **Integration tests** (`tests/`): Test end-to-end resolution behavior. Use `env::current_dir().unwrap().join("fixtures/integration")` to access fixtures.
- **Node.js tests** (`napi/`): Test the NAPI bindings with vitest.

#### Where to put fixtures

```
fixtures/
├── enhanced-resolve/      # Ported from webpack/enhanced-resolve (shared by many unit tests)
├── integration/           # Integration test fixtures (tests/ directory)
│   ├── misc/              # Unicode paths, BOM handling, package.json edge cases
│   ├── dot/               # Dot-path resolution
│   └── ...
├── dts_resolver/          # .d.ts resolution fixtures
├── invalid/               # Invalid configuration scenarios
├── pnp/                   # Yarn Plug'n'Play fixtures
├── pnpm/                  # pnpm node_modules structure
├── tsconfck/              # tsconfck compatibility
├── tsconfig/              # TypeScript config resolution
└── yarn/                  # Yarn monorepo fixtures
```

- Add new fixtures under the directory that matches the test file or feature area.
- For integration tests, add fixtures under `fixtures/integration/`.
- Ensure tests work on Windows, macOS, and Linux.

## Common Patterns

### Error Handling

```rust
// Use proper error types
use oxc_resolver::{ResolveError, ResolveErrorKind};

match resolver.resolve(path, specifier) {
    Ok(resolution) => { /* handle success */ },
    Err(ResolveError { kind: ResolveErrorKind::NotFound, .. }) => { /* handle not found */ },
    Err(err) => { /* handle other errors */ }
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oxc-project/oxc-resolver](https://github.com/oxc-project/oxc-resolver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
