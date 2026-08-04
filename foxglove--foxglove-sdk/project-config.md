---
trigger: always_on
description: Instructions for AI (and human) contributors.
---


# Foxglove SDK Codebase Rules

Instructions for AI (and human) contributors.

## About Foxglove

Foxglove is an observability and visualization platform for multimodal robotics data.
This repository is the Foxglove SDK: a multi-language SDK for logging and visualizing multimodal data.
The core SDK is written in Rust, with bindings for Python and C/C++, plus TypeScript schema definitions and ROS packages.

## Foundational Principles

- Preserve existing patterns, naming conventions, and project structure
- Minimize changes - modify only what is necessary for quality, maintainability, and the task at hand

## Repository Layout

| Directory                                     | Purpose                                                         |
| --------------------------------------------- | --------------------------------------------------------------- |
| `rust/foxglove`                               | Core Rust SDK crate                                             |
| `rust/foxglove_derive`                        | Derive macros for the Rust SDK                                  |
| `rust/foxglove_proto_gen`                     | Protobuf code generation for Rust                               |
| `rust/foxglove_data_loader`                   | Data loader crate (runs in app via WASM)                        |
| `rust/remote_data_loader_backend_conformance` | Conformance tests for remote data loader backends               |
| `rust/examples/`                              | Rust example programs                                           |
| `c/`                                          | C SDK (FFI layer built on top of Rust)                          |
| `cpp/foxglove`                                | C++ SDK                                                         |
| `cpp/foxglove_data_loader`                    | C++ data loader                                                 |
| `cpp/examples/`                               | C++ example programs                                            |
| `python/foxglove-sdk`                         | Python SDK (PyO3 bindings to Rust core)                         |
| `python/foxglove-sdk-examples`                | Python example programs                                         |
| `python/foxglove-schemas-flatbuffer`          | Flatbuffer schema definitions for Python                        |
| `python/foxglove-schemas-protobuf`            | Protobuf schema definitions for Python                          |
| `typescript/schemas`                          | TypeScript schema definitions                                   |
| `schemas/`                                    | Schema definitions (flatbuffer, jsonschema, omgidl, proto, ros) |
| `ros/`                                        | ROS message package                                             |
| `scripts/`                                    | Build and code generation scripts                               |
| `playground/`                                 | Interactive playground/examples                                 |

## Technology Stack

- **Rust** - Core SDK implementation; Cargo workspace at the repo root
- **Python** - PyO3 bindings; managed with `uv`; type-checked with `mypy`; formatted with `black` + `isort`; linted with `flake8`
- **C/C++** - FFI layer (C) and idiomatic wrapper (C++); built with CMake + `cargo` (via `corrosion`)
- **TypeScript** - Schema definitions, codegen, and CI scripts; managed with `yarn`; tested with `jest`
- **Schemas** - Protobuf, Flatbuffers, JSON Schema, OMG IDL, ROS 1/2 message definitions; generated via `make generate`
  - The schemas are defined in `typescript/schemas/src/internal/schemas.ts`

## High-level Architecture

- Context — binds channels to sinks; channels and sinks belong to exactly one context; the global context is used by default
- Channel — typed or untyped message stream on a topic; created once, reused for all messages on that topic
- Sink — receives logged messages; built-in sinks are McapWriter (file logging), WebSocketServer (live visualization), remote access Gateway (remote viz & teleop)
- The Python and C/C++ APIs are thin wrappers over this same model

## Development Guidelines

### Rust

- Prior to committing changes or considering them completed, for the modified rust project(s) run:
  `cargo test --features full`
  `cargo test --no-default-features`
  `cargo fmt`
  Run cargo check and clippy for the entire workspace:
  `cargo check --features full`
  `cargo clippy --no-deps --tests --features full -- -D warnings`
- Prefer `crate::` import over `super::` import; though importing `super::*` is fine within a test module (`mod tests`)
- Use `mod tests` rather than `mod test` for declaring unit tests in a submodule
- The MSRV (Minimum Supported Rust Version) is defined in Cargo.toml. Don't use Rust features that aren't stabilized as of this version.
- Use the tracing crate (tracing::info!, tracing::warn!, etc.), not println!, eprintln!, or the log crate macros directly
- Modules should be defined as `foo.rs`, not `foo/mod.rs`
- Use `cargo public-api` for evaluating public API changes

### Python


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [foxglove/foxglove-sdk](https://github.com/foxglove/foxglove-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
