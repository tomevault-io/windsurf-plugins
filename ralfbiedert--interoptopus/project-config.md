---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Interoptopus is a polyglot bindings generator for Rust libraries. It allows you to write a Rust library once and generate idiomatic bindings for multiple target languages (C#, C, Python). The core concept is using FFI-safe Rust code with special attributes (`#[ffi]`) that get processed into language-specific bindings.

## Development Commands

### Core Commands
- `just ci` - Run full CI locally (build, fmt check, clippy, test)
- `just build` - Build all workspace crates with all features.
- `just test` - Run all tests with all features via `cargo nextest`
- `just test-agent` - Can be used by agents to test a specific task
- `cargo check --workspace` - Quick check all crates

Agents may modify the `Justfile` section `test-agent` for their needs w.r.t the task at hand.


### Setup
- `just binstall-deps` - Install required tools (`cargo-insta`, `cargo-nextest`) via `cargo-binstall`


## Architecture

### Workspace Structure
```
crates/
├── core/                   # Main interoptopus crate with core FFI abstractions
├── proc_macros/            # Proc macro crate used by core
├── proc_macros_impl/       # Procedural macros (#[ffi_type], #[ffi_function]) incl. testing function
├── backend_csharp/         # Next-gen C# backend (experimental)
├── backend_cpython/        # Python bindings generation (defunct for now)
├── backend_c/              # C header generation (defunct for now)
├── backend_utils/          # Shared utilities for backends
└── reference_project/      # Comprehensive test project using all features
```

### Core Components

**Inventory System** (`crates/core/src/inventory/`): The central registry (`RustInventory`) that tracks all FFI types, functions, constants, and services via HashMaps keyed by unique IDs. Libraries build an inventory using a builder pattern with registration macros like `function!()`, `service!()`, `constant!()`, `extra_type!()`, `builtins_string!()`, `builtins_vec!()`.

**Proc Macros** (`crates/proc_macros_impl/src/`): The `#[ffi]` attribute is the unified entry point that dispatches based on item type — structs/enums go to `types::ffi()`, functions to `function::ffi()`, constants to `constant::ffi()`, and impl blocks to `service::ffi()`.

**FFI Patterns** (`crates/core/src/pattern/`): Higher-level abstractions built on primitives:
- `ffi::Slice<T>` / `ffi::SliceMut<T>` - Safe slice passing
- `ffi::Option<T>` - Optional values
- `ffi::Result<T, E>` - Error handling
- `ffi::CStrPtr` - ASCII strings
- `ffi::String` - UTF-8 strings
- `ffi::Vec<T>` - Growable arrays
- Service patterns - Turn Rust impl blocks into class-like interfaces

**Backends**: Transform the `RustInventory` into target language bindings. Each backend uses the Tera template engine with embedded templates and follows a builder pattern: `Interop::builder().inventory(inv).build()?.write_file("out.cs")?`.

**Reference Project** (`crates/reference_project/`): Comprehensive examples of every supported feature. Used for:
- Testing all backends work correctly
- Demonstrating best practices
- Validating new features
- Key modules:
  - `functions/` - All function patterns
  - `types/` - Type definitions
  - `patterns/` - Pattern implementations
  - `services/` - Service pattern examples
  - `constants.rs` - Constant exports


### New C# Backend (`backend_csharp`)

The next-gen C# backend uses a **multi-pass pipeline** architecture. 

The backend consists of these major items:

- A `lang` module that defines a base ontology of C# constructs the backend supports and can emit. It is the foundation of model passes and all entities there should be minimal and orthogonal to just express what is needed to define a C# FFI boundary.
- The `pass` module contains model and output passes. 
  - **Model passes** start with core library 'inventory' types and transform them to a well-defined data and interop model based C# `lang` items. Model passes encode the semantics of the interop files to be produced. 
    - Each model pass usually holds part of the transformation ('lang') model. The reason they are broken up into smaller units is to allow for incremental processing, make the logic easier to reason about, and allow for re-use of passes later for other codegen purposes.
    - Model passes should be 'reasonably sized'. If a model pass does or 'knows' too much it should be broken up into smaller passes. Overall passes should not be larger than a few pages of code.  
    - They are iterative and run until convergence (no more changes are made).
    - They can use other model passes by getting or setting data from them (preferably mostly getting), but should not depend on or reuse random helper functions from other passes.
  - **Output passes** take the model and render it into a final output format. Output passes render semantic models into actual, runnable C# code. 
    - They are primarily based on Tera templates
    - They are sequential and run after model passes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ralfbiedert/interoptopus](https://github.com/ralfbiedert/interoptopus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
