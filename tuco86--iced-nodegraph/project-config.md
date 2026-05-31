---
trigger: always_on
description: This document provides essential context for Claude Code when working on the iced_nodegraph project.
---

# Claude Code Instructions for iced_nodegraph

This document provides essential context for Claude Code when working on the iced_nodegraph project.

## Development Workflow

**Phases:**
1. **MVP** - Implement minimal working version of the feature
2. **Fix** - Address all observed errors and issues
3. **Refactor** - Improve code quality, structure, and readability
4. **Commit** - Once code is clean, create a commit
5. **Push** - Only after all checks pass

**Pre-Push Checklist (all must pass):**
- `cargo test -p iced_nodegraph` - unit tests
- `cargo check -p iced_nodegraph` - native compilation
- `cargo check -p iced_nodegraph --target wasm32-unknown-unknown` - WASM compilation
- `cargo clippy -p iced_nodegraph -- -D warnings` - lints
- `cargo build -p iced_nodegraph` - full build

A task is only complete when all checks pass and code is pushed.

## Automatic Validation

**Via SubagentStop hook:**
When a subagent/task completes, `.claude/hooks/validate.ps1` runs:
- `cargo check -p iced_nodegraph` - reports native compile errors
- `cargo test -p iced_nodegraph` - reports test failures

The script only outputs on errors to avoid filling context.

**Note:** Run `cargo fmt --all` manually before committing if desired.

Use the `code-reviewer` agent for reviewing significant code changes before committing.

## Git Commit Message Rules

**Format**: `type(scope): summary` (Conventional Commits)

**Types**: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `style`, `perf`

**Rules**:
- Single line only (no body unless explicitly requested)
- Summary max 60 characters
- Imperative mood: "add", "fix", "remove" (not "added", "fixed")
- Focus on WHY, not WHAT (intention over implementation details)
- No bullet lists, no file listings, no diff dumps

**Examples**:
- `feat(camera): add zoom-at-cursor transformation`
- `fix(wasm): resolve time platform incompatibility`
- `refactor: separate library from demo dependencies`
- `docs: clarify coordinate system formulas`

**Note**: Claude Code automatically adds co-author attribution when creating commits.

## Project Status

**Pre-Release**: This project has not been published to crates.io yet. No backwards compatibility is required - breaking API changes are acceptable.

## Documentation Standards

**CRITICAL**: Use minimal, professional language in all documentation:
- **NO EMOJIS** in code comments, documentation, or console output
- Use clear, technical language without informal expressions
- Status indicators: "VERIFIED", "TESTED", "INCOMPLETE" instead of emoji symbols
- Professional tone in all user-facing text and developer documentation

## Tool Usage Preferences

**ALWAYS use LSP (cclsp MCP) for Rust code navigation - it's faster and more accurate than grep:**

| Task | Tool | Example |
|------|------|---------|
| Find definition | `mcp__cclsp__find_definition` | `symbol_name: "NodeGraph"` |
| Find all usages | `mcp__cclsp__find_references` | `symbol_name: "edge_defaults"` |
| Rename symbol | `mcp__cclsp__rename_symbol` | `symbol_name: "old", new_name: "new"` |
| Get diagnostics | `mcp__cclsp__get_diagnostics` | `file_path: "src/lib.rs"` |

**Parameters:**
- `file_path`: File where symbol is defined (for context)
- `symbol_name`: Name of the symbol to find
- `symbol_kind`: Optional - "function", "struct", "method", "field", etc.

**When to use Grep/Glob instead:**
- Searching in string literals or comments
- Non-Rust files (toml, md, wgsl)
- Regex pattern searches
- LSP server unavailable

**Common patterns to follow:**
- When adding a new global config field, use `find_references` on `pin_defaults` to see the pattern
- When modifying NodeGraph API, check usages in demos with `find_references`

## Architecture Overview

This workspace contains a node graph editor built on Iced 0.14:

- **`iced_nodegraph`** - Custom node graph widget built on Iced GUI framework *(main project)*
- **`iced_sdf`** - Segment-based SDF renderer providing exact distance fields for nodes, edges, pins, and pin cutouts
- **`demos/*`** - hello_world, styling, interaction, 500_nodes, shader_editor, plus a shared `common` crate

`ngwa-rs` (a SpacetimeDB backend module) is an optional, separate sibling workspace at `../ngwa-rs`. It is NOT a member of this workspace's `Cargo.toml` and is not required to build or run the widget or demos.

**Dependencies**: Uses `iced = "0.14"` from crates.io and the in-tree `iced_sdf` crate for SDF-based rendering.

**Current Status**: Core functionality is complete - node/pin interaction, edge connections, and coordinate transformations are fully functional with type-safe API.

### WASM Browser Compatibility
- **Chrome/Chromium**: Full WebGPU support, recommended browser
- **Firefox**: WebGPU has known buffer-mapping issues (async timing bugs), may crash
- **Safari**: Untested

For WASM demos, Chrome or Chromium-based browsers are recommended.

## Core Architecture Patterns

### Coordinate System Abstraction - VERIFIED & TESTED
The project uses **euclid** crate for type-safe coordinate transformations:
- `WorldPoint`/`ScreenPoint` distinguish coordinate spaces with compile-time type safety
- `Camera2D` handles zoom/pan transformations in `src/node_graph/camera.rs`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tuco86/iced_nodegraph](https://github.com/tuco86/iced_nodegraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
