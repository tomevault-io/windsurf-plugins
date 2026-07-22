---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

DON'T COMMIT UNLESS EXPLICITLY ASKED TO DO SO BY THE USER! Previous commit requests do not matter - don't commit unless you've just been explicitly asked to do so.

## Project Overview

Monty is a sandboxed Python interpreter written in Rust. It parses Python code using Ruff's `ruff_python_parser` but implements its own runtime execution model for safety and performance. This is a work-in-progress project that currently supports a subset of Python features.

Project goals:

- **Safety**: Execute untrusted Python code safely without FFI or C dependencies, instead sandbox will call back to host to run foreign/external functions.
- **Performance**: Fast execution through compile-time optimizations and efficient memory layout
- **Simplicity**: Clean, understandable implementation focused on a Python subset
- **Snapshotting and iteration**: Plan is to allow code to be iteratively executed and snapshotted at each function call
- **Cross-platform**: Runs on Linux, macOS, and Windows (and any other OS that can run Rust)
- Targets the latest stable version of Python, currently Python 3.14

## `monty-types` — shared boundary types

The public data types (`MontyObject`, `MontyException`/`ExcType`, `OsFunctionCall` +
its arg structs, `ResourceLimits`/`ResourceTracker`, `PrintStream`/`PrintWriter`,
`CompileOptions`, `ExtFunctionResult`, `FileMode`, ...) live in `crates/monty-types`,
which depends on no other monty crate except the `monty-macros` derives. `monty`
depends on `monty-types` but does not blanket re-export it — only a few types
are re-exported inline where they appear in `monty`'s public API (e.g.
`run::CompileOptions`, `run_progress::{ExtFunctionResult, NameLookupResult}`).
Code needing `MontyObject`, `MontyException`, `OsFunctionCall`, etc. must
depend on `monty-types` directly.

Host-side crates (`monty-fs`, `monty-pool`, `monty-proto` without its `worker`
feature, `monty-python`, `monty-js`) MUST depend on `monty-types`, NOT `monty` —
this keeps the interpreter out of their binaries. Only the worker side
(`monty-runtime`, `monty-wasm-runtime`, `monty-proto` with `worker`) links the
interpreter. Don't add a `monty` dependency to a host-side crate; if it needs a
type, that type belongs in `monty-types`.

Interpreter-coupled methods on these types live in `monty` as `pub(crate)`
extension traits (`ExcTypeExt`, `MontyObjectExt`, `MontyTypeExt`, `StackFrameExt`,
`FileModeExt`, `BuiltinsFunctionsExt`, `ExtFunctionResultExt`) — import the trait
to call e.g. `ExcType::type_error(...)` or `MontyObject::new(value, vm)`.

## Cross-Platform Requirements

Monty must work identically on Linux, macOS, and Windows. Within the Monty sandbox,
paths always use POSIX/Linux-style forward slashes (`/`) regardless of the host OS.
The `MountTable` handles translating between virtual POSIX paths and host-native paths.

Key rules:
- **Virtual paths** are always POSIX-style (`/mnt/data/file.txt`), never Windows-style
- **Host paths** use `std::path::Path`/`PathBuf` which handles OS differences automatically
- Avoid `#[cfg(unix)]`-only code in the main crate — all features must work on all platforms
- Tests in `crates/*/tests/` should be cross-platform; use helper functions for
  OS-specific APIs like symlink creation (see `symlink_file`/`symlink_dir` in
  `crates/monty-fs/tests/fs_security.rs`)
- CI runs `cargo test -p monty --features memory-model-checks` and `cargo test -p monty-fs`
  on Linux, macOS, and Windows

## Important Security Notice

It's ABSOLUTELY CRITICAL that there's no way for code run in a Monty sandbox to access the host filesystem, or environment or to in any way "escape the sandbox".

**Monty will be used to run untrusted, potentially malicious code.**

Make sure there's no risk of this, either in the implementation, or in the public API that makes it more like that a developer using the pydantic_monty package might make such a mistake.

Possible security risks to consider:
* filesystem access
* path traversal to access files the users did not intend to expose to the monty sandbox
* memory errors - use of unsafe memory operations
* excessive memory usage - evading monty's resource limits
* infinite loops - evading monty's resource limits
* network access - sockets, HTTP requests
* subprocess/shell execution - os.system, subprocess, etc.
* import system abuse - importing modules with side effects or accessing `__import__`
* external function/callback misuse - callbacks run in host environment
* deserialization attacks - loading untrusted serialized Monty/snapshot data
* regex/string DoS - catastrophic backtracking or operations bypassing limits
* information leakage via timing or error messages
* Python/Javascript/Rust APIs that accidentally allow developers to expose their host to monty code

## Filesystem Mounts (`crates/monty-fs/`)

The `MountTable` allows mounting real host directories into the sandbox at virtual paths,
with configurable access modes (ReadWrite, ReadOnly, OverlayMemory).

Mounts are HOST-side code: the `monty` interpreter crate performs no filesystem
I/O and does not depend on `monty-fs`. Sandboxed code suspends with an

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pydantic/monty](https://github.com/pydantic/monty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
