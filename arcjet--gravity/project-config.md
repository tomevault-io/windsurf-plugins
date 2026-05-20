---
trigger: always_on
description: Gravity is a host code generator for WebAssembly Components. It takes a Core
---

# Gravity

Gravity is a host code generator for WebAssembly Components. It takes a Core
Wasm file (with embedded WIT custom section) and generates Go bindings targeting
[wazero](https://wazero.io/), a zero-dependency WebAssembly runtime for Go.

## Build & Run

### Prerequisites

- Rust toolchain (version pinned to what's in in `rust-toolchain.toml`)
- `wasm32-unknown-unknown` and `wasm32-wasip1` targets (configured in
  `rust-toolchain.toml`)
- Go (for running example tests)

### Building

```sh
cargo build
```

### Running

Gravity takes a Core Wasm file and produces Go source + an optimized `.wasm`
file:

```sh
cargo run --bin gravity -- --world <world-name> --output <output.go> <input.wasm>
```

## Testing

### Unit Tests

Run all unit tests (does NOT include snapshot/CLI tests):

```sh
cargo test
```

### Snapshot / CLI Tests

The CLI tests use `trycmd` for snapshot testing and require the example Wasm
files to be built first. They are excluded from the default `cargo test` run.

```sh
# 1. Build example Wasm files
cargo build -p example-basic --target wasm32-unknown-unknown --release
cargo build -p example-iface-method-returns-string --target wasm32-unknown-unknown --release
cargo build -p example-instructions --target wasm32-unknown-unknown --release
cargo build -p example-regressions --target wasm32-unknown-unknown --release

# 2. Run CLI snapshot tests
cargo test --test cli
```

To update snapshot expectations when output changes intentionally:

```sh
TRYCMD=overwrite cargo test --test cli
```

Snapshot files live in `cmd/gravity/tests/cmd/` (`.toml` for config, `.stdout`
and `.stderr` for expected output).

**Important**: The `trycmd` dependency has the `filesystem` feature disabled.
This means `TRYCMD=overwrite` can only update _existing_ `.stdout`/`.stderr`
files — it cannot create new ones. When adding a new snapshot test, you must
manually create the `.stdout` and `.stderr` files first (e.g. by redirecting
gravity's output), then `TRYCMD=overwrite` can keep them in sync afterward.

### Example Go Tests

The examples include Go test files that verify the generated bindings work
end-to-end with wazero. After building the example Wasm files and running
Gravity to regenerate bindings:

```sh
cd examples
go generate ./...
go test ./...
```

Or use `go generate` from the repo root (the `examples/generate.go` file has
`//go:generate` directives that build Wasm and run Gravity).

## Project Structure

```text
cmd/gravity/
  src/
    main.rs              # CLI entry point (clap argument parsing)
    lib.rs               # Library entry point
    codegen/
      mod.rs             # Codegen module root
      bindings.rs        # Top-level bindings generation (file structure, wasm embed)
      factory.rs         # Factory function codegen (instantiation boilerplate)
      imports.rs         # Import function analysis and host function codegen
      exports.rs         # Export function codegen (calling guest functions from Go)
      func.rs            # Instruction handler (canonical ABI instruction → Go code)
      ir.rs              # Intermediate representation types
      wasm.rs            # Wasm file processing and optimization
    go/
      mod.rs             # Go type representations module root
      type.rs            # Go type system (GoType enum, resolve_wasm_type)
      identifier.rs      # Go identifier naming conventions
      comment.rs         # Go comment formatting
      embed.rs           # Go embed directive generation
      imports.rs         # Go import path management
      operand.rs         # Operand type for code generation (Literal, SingleValue, MultiValue)
      result.rs          # GoResult type (Empty, Anon, Named)
  tests/
    cli.rs               # trycmd-based CLI snapshot test runner
    cmd/                  # Snapshot test data (*.toml, *.stdout, *.stderr)

examples/
  generate.go            # go:generate directives for building examples
  .gitignore             # Ignores generated *.go and *.wasm; keeps *_test.go
  basic/                 # Simple world with basic types
  iface-method-returns-string/  # Interface method returning a string
  instructions/          # Tests various canonical ABI instructions
  regressions/           # Regression tests for import codegen edge cases
```

**Note on generated files**: `examples/.gitignore` ignores all `*/*.go` files
(except `*/*_test.go`) and all `*/*.wasm` files. This means the generated Go
bindings and Wasm binaries are not committed — only the test files, WIT
definitions, and Rust source are tracked. You must run `go generate ./...` from
the `examples/` directory before the Go tests will compile.

## Architecture

### Key Concepts

- **WIT (WebAssembly Interface Types)**: The interface definition language for
  the Component Model. Gravity reads WIT embedded in Core Wasm files.
- **wit-bindgen-core**: Bytecode Alliance library (v0.53.1) that Gravity depends
  on for canonical ABI instruction generation. It provides the `Instruction`
  enum that `func.rs` handles.
- **Direction (Import vs Export)**: Gravity generates different code depending on
  whether a function is an import (Go host function called by Wasm guest) or an

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arcjet/gravity](https://github.com/arcjet/gravity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
