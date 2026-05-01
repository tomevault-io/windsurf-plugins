---
trigger: always_on
description: > A compile-time reactive web framework for Nim.
---

# Nexum — Agent Guide

> A compile-time reactive web framework for Nim.  
> Zero virtual DOM. Full SSR. Partial hydration.

---

## Project Overview

Nexum is an experimental frontend framework that uses Nim's macro system to perform at compile time what other frameworks do at runtime. It generates fine-grained DOM updates from reactive signals, produces server-rendered HTML with island hydration markers, and ships minimal JavaScript to the browser.

**Status:** Experimental / Design phase. Core signal runtime and the `buildHtml` macro compiler are in place. Full hydration engine is under active design.

**License:** GNU Affero General Public License v3.0 (AGPL-3.0)

**Required Nim version:** >= 2.0.0

---

## Technology Stack

- **Language:** Nim (>= 2.0.0)
- **Backends:** Dual-target — C backend for the server, JS backend for the client
- **Dependencies:** Pure Nim standard library only; no external Nimble dependencies
- **Key Nim features used:** Macros (`macro`), compile-time evaluation, `when defined(js)` conditional compilation, `{.experimental: "callOperator".}`

---

## Directory Structure

```
nexum/
  nexum.nimble              # Package manifest
  README.md                 # Human-facing quick start and philosophy
  ARCHITECTURE.md           # Full design document with roadmap
  AGENTS.md                 # This file
  src/
    nexum.nim               # Public API entry point — import this
    nexum/
      core/                 # Universal runtime (both backends)
        signals.nim         # Signal[T], Effect, Memo, batch(), untrack()
        scope.nim           # Scope, onCleanup, onMount, runInScope()
        context.nim         # Context, dependency injection, prop serialization
      compiler/             # Compile-time macro layer
        parser.nim          # buildHtml DSL → Intermediate Representation (IR)
        analyzer.nim        # Static/dynamic partitioning, dependency graph
        codegen_client.nim  # IR → DOM creation + signal effects (JS)
        codegen_server.nim  # IR → fast string builder (C / native)
        buildhtml.nim       # User-facing `buildHtml` macro
      runtime/              # Client JS runtime
        dom.nim             # Thin zero-dependency JS DOM wrappers
        hydrate.nim         # SSR marker scanning, island hydration
        patch.nim           # Mount/unmount helpers
      server/               # Server C runtime
        renderer.nim        # RenderContext, renderToString(), HTML escaping
        stream.nim          # Streaming SSR helpers
      router.nim            # Isomorphic router (client + server)
      app.nim               # Application bootstrap: initApp(), startClient(), startServer()
      macros.nim            # @component and @island decorators
  tests/                    # Unit and integration tests
    nim.cfg                 # Adds --path:"../src"
    test_signals.nim
    test_signals_equality.nim
    test_batch.nim
    test_ssr.nim
    test_buildhtml.nim
    test_buildhtml_js.nim
    test_router.nim
    test_compiler.nim
  site/                     # Official site (dog-food example)
    build.nim               # Static site builder → dist/
    serve.nim               # Async dev server
    index.nim               # Page components using buildHtml
    client.nim              # Client-side demo islands
```

---

## Build and Test Commands

```bash
# Run all tests
nimble test

# Build the official static site (outputs to dist/)
nimble site

# Serve the site locally for development (auto-picks port from 3000)
nimble serve

# Compile a single test manually
nim c -r tests/test_signals.nim

# Compile a client JS bundle manually
nim js --path:"src" -o:dist/client.js site/client.nim

# Compile server binary
nim c --path:"src" site/build.nim
```

Test files use either `std/unittest` or plain `assert`. The `tests/nim.cfg` adds `--path:"../src"` so tests can `import nexum` directly.

---

## Code Style Guidelines

Follow the existing conventions in the codebase:

- **Indentation:** 2 spaces.
- **Doc comments:** Use `##` for module-level documentation and public API symbols. Private helpers may use `#`.
- **Exports:** Public symbols are explicitly marked with `*` (e.g., `proc foo*()`).
- **Module guards:** Backend-only modules must start with a guard:
  ```nim
  when not defined(js):
    {.error: "dom.nim targets JS only".}
  ```
- **Call operator:** `signals.nim` enables `{.experimental: "callOperator".}` so signals can be read/written with `s()` and `s(value)`.
- **Backend split:** Use `when defined(js):` / `else:` for code that differs between client and server. Keep shared logic in the `core/` namespace.
- **Naming:**
  - Types: `PascalCase`
  - Procs/vars: `camelCase`
  - Constants: often `PascalCase` or `camelCase`
  - Private helpers: `camelCase`, no `*`

---

## Key Concepts and Patterns

### Signals and Reactivity

The reactive primitive is `Signal[T]`. Use `signal(value)` to create one. Effects auto-track signal reads.

```nim
let count = signal(0)
createEffect(proc() =
  echo count()  # auto-subscribes
)
count.set(1)    # triggers effect
```

- `s()` reads a signal (with dependency tracking).
- `s.get(track)` or `s.get(untrack)` for explicit control.
- `s.set(value)` writes a signal.
- `memo(proc(): T)` creates a computed memo.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nim-community/nexum](https://github.com/nim-community/nexum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
