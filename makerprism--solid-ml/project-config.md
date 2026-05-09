---
trigger: always_on
description: **NEVER use opam commands or access .opam or _opam folders, except inside `examples/ssr_api_app/Dockerfile` when installing Melange for the Docker build.**
---

# Agent Guidelines for solid-ml-server

## Hard Constraints

**NEVER use opam commands or access .opam or _opam folders, except inside `examples/ssr_api_app/Dockerfile` when installing Melange for the Docker build.**

This project builds with `/home/sabine/.opam/landing-pages-build/bin/dune` (dune 3.20.2) which is configured to work with the system OCaml installation. Do not use `opam` commands or reference any packages in `.opam/` or `_opam/` directories, except for the Dockerfile exception above. Always use the dune binary specified in the Makefile's `DUNE` variable.

**Git workflow:** Always use rebase (not merge) when integrating remote changes before pushing.

**Branch/PR workflow:** Work in a feature branch (not `main`) and open a pull request for each session's feature/fix. Do not stack PRs: keep exactly one active PR per session and wait for it to be merged (or explicitly closed) before starting another PR branch.

**Examples policy:** When adding a user-facing feature, update all relevant examples in the same session/PR to demonstrate the new capability. Do not defer example updates to a later stacked PR.

## Product Stage Policy

This project is still in an early stage. Prioritize DX and simplicity over API stability.

- Prefer simple, obvious, ergonomic solutions over complex compatibility layers.
- It is acceptable to change APIs when it meaningfully improves usability.
- Do not block improvements for backward compatibility concerns at this stage.
- When you introduce a breaking change, update docs/tests in the same change so migration is clear.

## Project Overview

solid-ml-server is an OCaml framework for building reactive web applications with SSR, inspired by SolidJS.

**Repository:** github.com/makerprism/solid-ml  
**License:** MIT

## Documentation

- **Architecture:** `docs/A-01-architecture.md` - Full design document with API specs, SSR flow, and development phases

## Package Structure

| Package | Purpose | Status |
|---------|---------|--------|
| `solid-ml-internal` | Shared reactive core (functor-based, DO NOT use directly) | Complete |
| `solid-ml-server` | Server-side reactive framework (OCaml 5 with DLS) | Complete |
| `solid-ml-ssr` | Server-side rendering to HTML strings | Complete |
| `solid-ml-browser` | Browser-side reactive framework (Melange) | Complete |
| `solid-ml-router` | SSR-aware routing with data loaders | Complete |

## Build Commands

```bash
# Build all packages
dune build

# Run tests
dune runtest

# Clean build artifacts
dune clean

# Build browser examples (requires Node.js for esbuild bundling)
make browser-examples

# Run browser tests
make browser-tests
```

## Architecture

### Shared Core (`solid-ml-internal`)

The project uses a **functor-based architecture** to share reactive algorithms between server and browser:

```
solid-ml-internal/
├── types.ml           # Internal types (Obj.t for type erasure, phantom types for safety)
├── backend.ml         # Backend.S module type + Backend.Global implementation
├── reactive_functor.ml # Make(Backend) functor with all reactive algorithms
└── solid_ml_internal.ml
```

**Key insight:** The only platform difference is how the current runtime is stored:
- Server: Domain-local storage (thread-safe isolation per domain)
- Browser: Global ref (safe in single-threaded JS)

### Server (`solid-ml-server`)

```ocaml
(* Defines DLS backend and instantiates functor *)
module Backend_DLS : Internal.Backend.S = struct
  let runtime_key = Domain.DLS.new_key (fun () -> None)
  let get_runtime () = Domain.DLS.get runtime_key
  let set_runtime rt = Domain.DLS.set runtime_key rt
  let handle_error exn _ = raise exn
end

module R = Internal.Reactive_functor.Make(Backend_DLS)
```

### Browser (`solid-ml-browser`)

```ocaml
(* Defines global ref backend with console.error *)
module Backend_Browser : Internal.Backend.S = struct
  let current_runtime = ref None
  let get_runtime () = !current_runtime
  let set_runtime rt = current_runtime := rt
  let handle_error exn context =
    console_error ("solid-ml-server: Error in " ^ context ^ ": " ^ Printexc.to_string exn)
end

module R = Internal.Reactive_functor.Make(Backend_Browser)
```

### Type Safety

The internal types use `Obj.t` for type erasure (needed for heterogeneous collections), but expose type-safe APIs via phantom types:

```ocaml
(* Internal: untyped *)
type signal_state = { mutable value: Obj.t; ... }

(* Public: typed via phantom type parameter *)
type 'a signal = signal_state  (* 'a is phantom *)

let create_typed_signal (type a) ?equals (initial : a) : a signal =
  { value = Obj.repr initial; ... }

let read_typed_signal (type a) (s : a signal) : a =
  Obj.obj s.value
```

## Current Development Phase

**Phase 1: Reactive Core** (complete)

- [x] Signal.create, get, set, update, peek
- [x] Dependency tracking via execution context
- [x] Effect.create with auto-tracking
- [x] Effect.create_with_cleanup
- [x] Effect.untrack
- [x] Memo.create, create_with_equals (eager evaluation like SolidJS)
- [x] Batch.run with Signal integration
- [x] Owner.create_root, run_with_owner, on_cleanup, dispose
- [x] Context.create, provide, use

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [makerprism/solid-ml](https://github.com/makerprism/solid-ml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
