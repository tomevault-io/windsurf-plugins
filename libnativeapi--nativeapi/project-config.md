---
trigger: always_on
description: This is the workspace repo (`libnativeapi/nativeapi`, formerly `nativeapi-flutter` and `nativeapi-workspace`) for the [libnativeapi](https://github.com/libnativeapi) project family. Every binding (`bindings/dart/`, `bindings/rust/`, `bindings/csharp/`, `bindings/js/`, `bindings/python/`), the code generator (`tools/codegen/`), the `./codegen` script, the specs and the shared tooling live directly in this repo (the Rust and C# histories were merged in from `nativeapi-rust` and `nativeapi-csharp`)
---

# libnativeapi workspace

This is the workspace repo (`libnativeapi/nativeapi`, formerly `nativeapi-flutter` and `nativeapi-workspace`) for the [libnativeapi](https://github.com/libnativeapi) project family. Every binding (`bindings/dart/`, `bindings/rust/`, `bindings/csharp/`, `bindings/js/`, `bindings/python/`), the code generator (`tools/codegen/`), the `./codegen` script, the specs and the shared tooling live directly in this repo (the Rust and C# histories were merged in from `nativeapi-rust` and `nativeapi-csharp`); only `core/` is a git submodule of an independent repository. Work inside `core/` is committed and pushed from that subdirectory; everything else is committed here.

## Layout

```
core/               # submodule: nativeapi-core — the C++ core library
bindings/
├── dart/           # the Dart binding: nativeapi/, cnativeapi/, nativeapi_flutter/
├── rust/           # the Rust binding: crates/{nativeapi,cnativeapi}
├── csharp/         # the C# binding: src/, tests/, NativeAPI.slnx
├── js/             # the JS/TS binding: a Node-API addon (src/) + TypeScript (lib/)
└── python/         # the Python binding: ctypes package (nativeapi/) + native shim (src/)
examples/           # every binding's example apps, prefixed dart_*, flutter_*, rust_*, csharp_*, js_*, python_*
pubspec.yaml        # pub workspace + melos root: Dart packages and Flutter examples
Cargo.toml          # cargo workspace root: Rust crates and examples
tools/codegen/      # in-repo Rust workspace: the code generator
tools/gui/          # GUI tests and demo scenarios for the examples (built on the skills)
codegen             # Python entry point orchestrating the generators
.agents/skills/     # agent skills: core API changes, GUI testing, demo recording (see below)
.claude/skills      # symlink → ../.agents/skills, so Claude Code discovers the same skills
```

## Architecture

- `core` — the C++ core library (repo: `nativeapi-core`). The source of truth for the native API surface (windows, tray icons, menus, displays, keyboard, dialogs, storage, etc.) with per-platform implementations (macOS/Windows/Linux).
- `tools/codegen` — three crates: `shared` (libclang parser, IR, naming), `capi` (C ABI + umbrella header), `bindings` (Rust/Dart/C#/JS/Python generators, consuming the IR JSON emitted by `capi`). Only `capi` depends on libclang. See tools/codegen/README.md.
- `bindings/*` — language bindings wrapping the core library. All live in this repo and build against the `core/` submodule directly (Rust `build.rs`, the Dart `cnativeapi` package's build hook, the C# native CMake, the JS addon's and the Python binding's `CMakeLists.txt`). Only a published package carries its own copy of core, in `cxx_impl/`, which the release workflows vendor and never commit. The Rust binding layers `nativeapi` (safe API) over `cnativeapi` (FFI). The Python binding has no compiled extension: generated `ctypes` code (`nativeapi/_capi.py` plus one module per header) calls a shared library built from core and a small event loop shim, which `Application.run_async()` pumps from asyncio.

## Design specs

`specs/` holds the settled design rules for `core/` — layering, the identity/value object
model, the public API style, the platform seam, the event system, managers, and the C ABI.
Start at [specs/README.md](specs/README.md); read the relevant spec before adding or
reshaping public API in `core/src/`.

Any diff that touches a public header in `core/src/` must pass the checklist at the end of
[specs/api-style.md](specs/api-style.md) — naming vocabulary, parameter and return types,
failure reporting, platform-availability notes, and the codegen constraints. When existing
headers disagree with each other, follow the spec, not the nearest neighbour: it records
which side of each split is the rule and which is legacy.

There is no separate issue list: each spec carries the open questions and known legacy
gaps of its own area inline (an "未决" section, or a "存量缺口" note next to the rule it
breaks). When one is resolved, edit the spec text itself.

## Code generation

Always drive the generators through `./codegen` at the workspace root:

- `./codegen` — full run: C ABI, then all bindings
- `./codegen capi` / `./codegen bindings [--lang rust,dart,csharp,js,python]`
- `./codegen check` — read-only verification, non-zero exit when stale (CI mode)
- `./codegen readme` — copy the shared README sections (`tools/readme/*.md`, e.g. Contributing) into core and every binding; `check` flags drift, `sync` runs it. Edit the snippet, never the copies.
- `./codegen sync [-m "msg"] [--push]` — full downstream propagation, see below


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [libnativeapi/nativeapi](https://github.com/libnativeapi/nativeapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
