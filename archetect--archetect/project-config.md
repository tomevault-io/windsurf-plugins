---
trigger: always_on
description: **This is the Archetect 3 repository** — a major version evolution of Archetect, the code-centric, language-agnostic code generator written in Rust.
---

# CLAUDE.md

## Project Overview

**This is the Archetect 3 repository** — a major version evolution of Archetect, the code-centric, language-agnostic code generator written in Rust.

- **v3 repo**: https://github.com/archetect/archetect-3
- **v2 repo** (stable, production): https://github.com/archetect/archetect
- **Docs** (v2): https://archetect.github.io
- **GitHub Org**: https://github.com/archetect
- **License**: MIT

The codebase currently reflects v2.1.0. v3 work builds on top of it.

## v3 Initiatives

Three planned initiatives, documented in `docs/plans/`:

1. **[IO Protocol Overhaul](docs/plans/archetect-3-io-overhaul.md)** — Route file writes through the IO channel (not direct `std::fs`). Introduce `ScriptIoHandle`/`ClientIoHandle` traits, fallible error handling, WriteFile/WriteDirectory/Complete/Ack/Initialize messages. Enables the closed-source CodegenExtension for COS.

2. **[Lua Scripting Engine](docs/plans/archetect-3-lua-scripting-engine.md)** — Add Lua (via mlua) as the primary scripting engine with a redesigned v3 API. Rhai retained as a frozen compatibility layer for v2 archetypes. The Lua API is not a port of Rhai — it's a clean-slate redesign: `Context` object, dedicated prompt methods (`ctx:text()`, `ctx:select()`), simplified case system (`Cases.programming()`), namespaced modules (`git.init()`, `shell.run()`), LuaLS annotations for full IDE support.

3. **[Warts and Improvements](docs/plans/archetect-3-warts-and-improvements.md)** — Comprehensive catalog of v2 issues: 25+ panic paths via `.unwrap()`, lost script error context, missing manifest validation, no dry-run, no test framework, no component versioning, documentation gaps. Prioritized fix list for v3.

## Related Projects

- **v2 codebase**: `/Users/jimmie/personal/archetect/archetect` — stable production CLI, do not mix with v3 work
- **Production archetypes**: `/Users/jimmie/work/p6m-archetypes` — ~80 Rhai archetypes, the backwards-compat benchmark
- **COS / Onyx**: `/Users/jimmie/personal/jimmiebfulton/onyx` — the CodegenExtension target platform
- **feature/client-server** (v2 branch): Has working gRPC proof-of-concept with the richer IO protocol. Reference implementation for the IO overhaul — don't port verbatim, but use as a guide

## Workspace Structure

Cargo workspace with 9 crates. Dependency graph:

```
archetect-bin (CLI entry point)
├── archetect-core (business logic, scripting, rendering)
│   ├── archetect-api (IoDriver trait, command types)
│   ├── archetect-templating (vendored MiniJinja 0.30.6)
│   ├── archetect-terminal-io (terminal IoDriver impl)
│   │   └── archetect-terminal-prompts (vendored inquire fork)
│   ├── archetect-inflections (case conversions, pluralization)
│   └── archetect-validations (validation error types)
└── xtask (build/install automation)
```

### Crate Purposes

| Crate | Purpose |
|-------|---------|
| `archetect-bin` | CLI (clap), configuration loading (figment), subcommand dispatch |
| `archetect-core` | Archetype/catalog loading, Rhai engine, template rendering, source/cache management |
| `archetect-api` | `ScriptIoHandle`/`ClientIoHandle` traits, prompt/write command structs |
| `archetect-templating` | Vendored MiniJinja — do not update from upstream without care |
| `archetect-terminal-io` | `TerminalIoDriver` bridging prompts to `ScriptIoHandle` |
| `archetect-terminal-prompts` | Vendored inquire fork — interactive terminal prompts |
| `archetect-inflections` | String transforms: camelCase, snake_case, plural/singular, etc. |
| `archetect-validations` | Validation rules and error types |
| `xtask` | `cargo xtask install` with optional `--static-openssl` |

## Build & Development Commands

```bash
# Build
cargo build

# Run tests (entire workspace)
cargo test

# Test specific crate
cargo test -p archetect-core
cargo test -p archetect-templating

# Install CLI locally for manual testing
cargo xtask install

# Run CLI without installing
cargo run -p archetect-bin -- <args>

# Lint and format
cargo clippy --all-targets --all-features
cargo fmt

# Build requires protoc (for gRPC proto compilation in archetect-core/build.rs)
# macOS: brew install protobuf
# Ubuntu: sudo apt install protobuf-compiler
```

## Architecture Deep Dive

### Core Flow: Rendering an Archetype

1. CLI parses args → loads Configuration (merging YAML config + CLI overrides via figment)
2. `Source` resolves the archetype origin (local path or Git URL with caching)
3. `Archetype` loads `archetype.yaml` manifest and validates requirements
4. Rhai engine executes `main.rhai` (or manifest-specified script)
5. Script calls `prompt()` → messages sent via `ScriptIoHandle` → terminal responds
6. Script calls `render_directory()` or `render()` → MiniJinja processes templates
7. Output files written to destination directory

### Key Source Files

| File | What it does |
|------|-------------|
| `archetect-bin/src/main.rs` | CLI entry point, arg parsing, subcommand dispatch |
| `archetect-core/src/archetect/archetect.rs` | `Archetect` struct — main orchestrator, builder pattern |
| `archetect-core/src/archetype/archetype.rs` | `Archetype` — loading, rendering, script execution |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [archetect/archetect](https://github.com/archetect/archetect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
