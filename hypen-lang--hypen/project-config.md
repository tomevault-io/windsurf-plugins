---
trigger: always_on
description: Hypen is a declarative UI language and runtime for building cross-platform applications. UI components are purely declarative while modules manage reactive state and business logic.
---

# AGENTS.md

## Project Overview

Hypen is a declarative UI language and runtime for building cross-platform applications. UI components are purely declarative while modules manage reactive state and business logic.

## Repository Structure

```
hypen-engine-rs/
├── parser/                      # Rust parser (Chumsky combinators)
├── hypen-engine-rs/             # Core reactive engine (Rust -> WASM)
├── hypen-sdk-rs/                # Rust SDK for embedding Hypen
├── hypen-web/                   # TypeScript/Bun Web SDK
│   ├── packages/core/           # @hypen-space/core - Platform-agnostic runtime (types, app, state, router - NO WASM)
│   ├── packages/web/            # @hypen-space/web - DOM & Canvas renderers (pure patch consumers)
│   ├── packages/server/         # @hypen-space/server - Node.js WASM engine, loader, discovery, RemoteServer
│   └── packages/web-engine/     # @hypen-space/web-engine - Browser WASM engine, Hypen orchestrator
├── hypen-cli/                   # @hypen-space/cli - CLI tools (init, dev, build)
├── hypen-docs/                  # Fumadocs/Next.js documentation site
├── hypen-renderer-android/      # Android/Kotlin native renderer
├── hypen-renderer-swift/        # iOS/SwiftUI native renderer
├── hypen-server-swift/          # Swift server SDK
├── hypen-kotlin/                # Kotlin/JVM SDK (module system with Kotlin DSL)
├── hypen-golang/                # Go implementation (module system)
├── hypen-lsp/                   # Language Server Protocol implementation
├── tailwind-parse/              # Tailwind CSS class parser (Rust)
├── component-gallery-server/    # Component gallery & screenshot testing
├── engine-compatibility-tests/  # Cross-SDK compatibility test suite
├── examples/                    # Example projects
├── skills/                      # Codex skills for Hypen development
└── scripts/                     # Build & utility scripts
```

## Development Commands

### Parser (Rust)
```bash
cd parser
cargo test                                 # Run all tests
cargo test test_name -- --nocapture        # Specific test with output
cargo run --example pretty_errors          # Pretty error display with Ariadne
```

### Engine (Rust/WASM)
```bash
cd hypen-engine-rs
cargo test                                 # Run tests natively
cargo clippy                               # Lint
./build-wasm.sh                            # Build all WASM targets (bundler, nodejs, web)
```

### Web SDK (TypeScript/Bun)
```bash
cd hypen-web
bun test                                   # Run tests
bun run playground                         # Interactive dev playground
bun run replayground                       # Rebuild WASM + restart playground
```

### CLI
```bash
cd hypen-cli
bun bin/hypen.ts init my-app               # Create new project
bun bin/hypen.ts dev                       # Dev server with hot reload
bun bin/hypen.ts build                     # Production build
```

### Docs
```bash
cd hypen-docs
bun install && bun dev                     # Local dev server
bun run build                              # Production build
```

### Kotlin SDK
```bash
cd hypen-kotlin
./gradlew test                             # Run tests (includes compatibility tests)
```

## Architecture

### Data Flow
```
Hypen DSL -> Parser (AST) -> Engine (IR) -> Reconciler -> Patches -> Platform Renderer
                                  ↑
                             State/Actions
```

### Core Components

**Parser** (`parser/`) - Rust combinator-based parser using Chumsky
- Entry: `src/lib.rs` exports `parse_component()`, `parse_components()`, `parse_document()`, `parse_import()`
- AST types: `src/ast.rs`
- Parser logic: `src/parser.rs`
- Tests: `src/tests.rs` (114 tests)

**Engine** (`hypen-engine-rs/`) - Core reactive runtime
- Orchestrator: `src/engine.rs` - `Engine` struct coordinates all systems
- WASM bindings: `src/wasm/` - `WasmEngine` in `js.rs`, WASI interface in `wasi.rs`
- IR system: `src/ir/` - AST to intermediate representation
- Reactive system: `src/reactive/` - dependency tracking for `@{state.*}` bindings
- Reconciliation: `src/reconcile/diff.rs` - keyed diffing, generates minimal patches
- Actions/Events: `src/dispatch/` - routes `@actions.*` to module handlers
- Lifecycle: `src/lifecycle/` - module/component lifecycle
- State: `src/state.rs` - observable state with change detection
- Serialization: `src/serialize/` - Remote UI protocol

**Web SDK** (`hypen-web/packages/`)
- `core/src/app.ts` - fluent API for defining stateful modules
- `core/src/state.ts` - Proxy-based reactive state
- `core/src/types.ts` - shared types (Patch, Action, etc.)
- `web/src/dom/` - browser DOM renderer
- `web/src/canvas/` - Canvas 2D renderer
- `server/src/engine.ts` - Node.js WASM engine wrapper
- `server/src/discovery.ts` - component file discovery
- `server/src/loader.ts` - dynamic component loader
- `web-engine/src/engine.ts` - browser WASM engine wrapper
- `web-engine/src/hypen.ts` - Hypen orchestrator class

**CLI** (`hypen-cli/`)
- `bin/hypen.ts` - command dispatcher
- `src/dev.ts` - dev server with hot reload

### Key Data Structures

**AST (Parser Output):**
```rust

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hypen-lang/hypen](https://github.com/hypen-lang/hypen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
