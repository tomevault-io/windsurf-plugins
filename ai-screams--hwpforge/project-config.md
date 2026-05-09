---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project Overview

HwpForge is a Rust library for programmatic control of Korean HWP/HWPX document formats, designed with LLM-first principles. The goal is to enable AI agents (like Claude Code) to generate Korean government proposal documents using natural language + Markdown + YAML style templates.

**Current Status**:

- HWPX codec: read/write shipped
- Markdown bridge: read/write shipped
- HWP5 converter path: active with style/layout fidelity line in progress
- CLI bindings: shipped
- MCP bindings: shipped
- Python bindings: stub
- Shared tab semantics: landed on `main`
- Shared `ordered / bullet / outline` semantics: implemented on local `feat/list-shared-semantics`
- Checkable bullet semantics: implemented on local `feat/list-shared-semantics`
- HWP5 checkable support: definition-level parity only; paragraph item checked-state decode is still backlog
- Markdown task lists normalize to HWPX-first checkable semantics; ordered task lists intentionally lose numbering
- HWP5 char/para style bridge now preserves the main supported style surface
- HWP5 layout hint patch injects `linesegarray` and safe table height hints for better visual parity
- `convert-hwp5` / `audit-hwp5` warning counts are aligned for style projection fallbacks
- HWP5/HWPX char effects now preserve `emboss`, `engrave`, `superscript`, and `subscript`
- Known style-fidelity gaps still deferred: `breakLatinWord=HYPHENATION`, richer strike/underline line families

**Workspace Facts (code-grounded)**:

- Cargo packages: `10`
- Workspace version: `0.5.0`
- Tracked Rust `src` files under `crates/`: `144`
- Tracked Rust `src` LOC under `crates/`: `90,629`
- Example artifact files under `examples/`: `67`
- GitHub workflow files: `5`
- MSRV: `1.88`
- Dev toolchain: Rust `1.93`

Treat these as code-derived facts, not roadmap promises.

---

## Architecture (Forge Metaphor)

The codebase follows a **blacksmith workshop** metaphor with clear separation of concerns:

```
Foundation (🔩 primitives)
  → Core (🔨 pure document structure, no style definitions)
  → Blueprint (📐 YAML style templates, centralized like Figma Design Tokens)
  → Smithy (🔥 format-specific compilers: HWPX, HWP5, Markdown)
  → Bindings (🐍⚒️🤖 Python/CLI/MCP interfaces)
```

**Key Principle**: **Structure and Style are separate** (like HTML + CSS).

- Core contains document structure with style **references** (IDs only)
- Blueprint contains style **definitions** (fonts, sizes, colors)
- Smithy compilers fuse Core + Blueprint → final format

This enables:

- One YAML template applied to multiple documents
- Format-agnostic document manipulation
- Easy addition of new formats (smithy-odt, smithy-pdf, etc.)

---

## Development Commands

### Build & Test

```bash
cargo build --workspace
cargo nextest run --workspace --all-features
cargo test -p hwpforge-foundation
make test
make ci-fast
make ci-full
```

### Lint & Format

```bash
cargo clippy -p hwpforge-foundation -- -D warnings
make clippy
make fmt
make fmt-fix
```

### Watch Mode

```bash
bacon         # Auto-run clippy on file changes
bacon test    # Auto-run tests
```

### Documentation & Coverage

```bash
make doc
make cov
```

---

## Crate Dependency Graph

```
foundation (NO dependencies except serde/thiserror)
    ↓
core (foundation only)
    ↓
blueprint (foundation + core)
    ↓
smithy-hwpx, smithy-hwp5, smithy-md (foundation + core + blueprint)
    ↓
bindings-py, bindings-cli, bindings-mcp (all smithy crates)
```

**Important**: Foundation is the root. If you modify foundation, ALL crates rebuild. Keep it minimal.

---

## Critical Design Patterns

### Working Principles

- **Warning-first for unknowns**: if source truth is missing or a value is unsupported, emit a warning or validation signal first.
- **No fake support**: do not silently normalize unknown semantics into arbitrary defaults just to keep output green.
- **Shared-model first**: if HWP5 discovers a semantic that Core/HWPX cannot carry, extend the shared representation first and wire HWP5 after.
- **Semver-first for public API**: if a design touches public structs, enums, or externally constructible types, surface the breakage before implementation and get approval first.

### 1. Color is BGR (NOT RGB!)

```rust
// ❌ WRONG — This is BLUE in BGR!
Color::from_raw(0xFF0000)

// ✅ CORRECT — red → 0x0000FF internally
Color::from_rgb(255, 0, 0)
```

HWP format uses BGR (Blue-Green-Red) byte order. Always use `from_rgb()` constructor.

### 2. HwpUnit Integer-Based Units

```rust
HwpUnit::from_pt(12.0)  // 12pt → HwpUnit(1200)
// 1pt = 100 HWPUNIT, 1mm ≈ 283 HWPUNIT
```

Integer-based to avoid floating-point precision errors. Valid range: ±100M.

### 3. Branded Index Types

```rust
CharShapeIndex::new(0)   // ✅ OK
let idx: ParaShapeIndex = CharShapeIndex::new(0);  // ❌ Compile error!
```

`Index<T>` uses phantom types. Cannot mix char/para/font indices.

### 4. Typestate Pattern (Core)

```rust
let doc = Document::<Draft>::new();
// doc.save_hwpx(...);  // ❌ Compile error! Draft cannot be saved
let validated = doc.validate()?;
// validated.save_hwpx(...);  // ✅ OK
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ai-screams/HwpForge](https://github.com/ai-screams/HwpForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
