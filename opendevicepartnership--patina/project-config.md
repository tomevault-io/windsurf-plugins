---
trigger: always_on
description: Patina is a Rust-based UEFI firmware project. It currently replaces the traditional C-based EDK II
---

# Patina Project Instructions

Patina is a Rust-based UEFI firmware project. It currently replaces the traditional C-based EDK II
DXE Core with a Rust implementation that introduces a component-based architecture with dependency
injection in addition to a general-purpose UEFI Rust SDK.

## Project Structure

The workspace is organized into four top-level areas:

- `components/` - Feature components (ACPI, Advanced Logger, MM, Performance, SMBIOS, etc.)
- `core/` - Shared core internals (debugger, collections, CPU, dependency expressions, stack traces)
- `patina_dxe_core/` - Main DXE Core library that ties everything together
- `sdk/` - Public SDK (`patina` crate for Boot/Runtime Services, component infrastructure, `patina_ffs` for firmware
   file system, `patina_macro` for proc-macros)

Crate dependency rules are strict. Components depend on `sdk/` only - never on each other or on `core/`. The SDK
depends only on generic external crates. Core crates may depend on each other and on the SDK. See
`docs/src/dev/code_organization.md` for the full dependency matrix.

## Build and Test Commands

All commands go through `cargo make`. Never run raw `cargo` commands.

- `cargo make check` - Type-check all code
- `cargo make fmt` - Format code (run after every edit)
- `cargo make clippy` - Lint with clippy
- `cargo make test` - Run all unit tests
- `cargo make all` - Full PR readiness (fmt-check, deny, cspell, clippy, build, test, coverage, doc)
- `cargo make build-x64` / `cargo make build-aarch64` - Build for UEFI targets
- `cargo make coverage` - Generate test coverage reports
- `cargo make doc` - Build documentation

See `Makefile.toml` for the complete command list.

## Module Organization

- Never use `mod.rs` files. Use named module files (e.g., `my_module.rs`) instead.
- No public definitions directly in `lib.rs` - only public module declarations.
- Break implementations into logical files with clean namespaces.
- Crate naming: `patina_` prefix for public crates, `patina_internal_` for internal
  crates, `_macro` suffix for proc-macro crates.

```rust
// WRONG: using mod.rs
// src/memory/mod.rs

// CORRECT: using named module file
// src/memory.rs
```

See `docs/src/component/requirements.md` for crate layout standards.

## Safety Conventions

- Prefer `zerocopy` for binary data and memory layouts over manual unsafe pointer/slice
  operations. `zerocopy` provides safe, zero-cost abstractions for interpreting byte
  buffers as typed data.
- Minimize `unsafe` code. Constrain it within safe abstraction wrappers.
- Document preconditions, postconditions, and invariants for every `unsafe` block.
- Prefer fallible constructors returning `Result` over constructors that silently
  handle invalid input or panic.
- Mark functions `unsafe` only when the caller must uphold a contract the function
  itself cannot verify internally. If the function validates all inputs before the
  unsafe operation, it can remain safe.

See `docs/src/dev/principles/unsafe.md` for the full safety philosophy
(software safety vs. hardware safety distinctions).

## Error Handling

- Prefer `Result` return types. Avoid panics in production code.
- At UEFI ABI boundaries (`extern "efiapi"` functions), use `efi::Status`.
- Internally, use domain-specific Rust error types with `Debug`, `Display`, and `Error`
  trait implementations.
- Implement `From<>` conversions at error type boundaries to create clean error chains.
- Use `expect("descriptive message")` over bare `unwrap()`. Reserve `unwrap()` for
  test code only.

See `docs/src/dev/principles/error-handling.md` for error
propagation patterns and examples.

## Logging

- Use the `log` crate for structured logging.
- Log at appropriate levels: `trace`, `debug`, `info`, `warn`, `error`.
- Include relevant context in log messages.
- Do not log in hot paths or have entry/exit logs.
- Prefer using `patina::writelncrlf` for formatting strings that will be logged over `writeln!`.

## Component Model

Patina uses dependency injection through component entry point function signatures.
A component only executes when all its declared dependencies are available.

- Define components with the `#[component]` attribute macro on an `impl` block.
  The impl must contain `fn entry_point(self, ...) -> Result<()>`.
- Register service implementations with `#[derive(IntoService)]` and `#[service(dyn Trait)]`.
- Param types: `Config<T>`, `ConfigMut<T>`, `Service<T>`, `Hob<T>`, `Commands`,
  `Handle`, `StandardBootServices`, `StandardRuntimeServices`, `&Storage`/`&mut Storage`,
  `Option<P>`, tuples.
- `ConfigMut<T>` components run first (config is unlocked); calling `lock()` makes the
  value immutable and enables `Config<T>` components to execute.
- `Service<dyn Trait>` is preferred over `Service<ConcreteType>` for mockability. Use
  a concrete wrapper struct only when generic method signatures are needed.
- Use the **stored dependencies** pattern: store all dependency references as fields in
  the component struct. The entry point stores references; methods use them.

See `docs/src/component/interface.md` and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenDevicePartnership/patina](https://github.com/OpenDevicePartnership/patina) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
