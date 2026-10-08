---
trigger: always_on
description: `rive-cli` is a Rust CLI and library that generates Rive `.riv` binary animation files from high-level AuthoringSpec or explicit JSON SceneSpec. It implements the **write side** of the Rive binary format; the read side is open-source. The tool supports programmatic creation of vector animations, state machines, and interactive graphics without the Rive editor.
---

# Repository Guidelines

## Project Overview

`rive-cli` is a Rust CLI and library that generates Rive `.riv` binary animation files from high-level AuthoringSpec or explicit JSON SceneSpec. It implements the **write side** of the Rive binary format; the read side is open-source. The tool supports programmatic creation of vector animations, state machines, and interactive graphics without the Rive editor.

**Primary commands**: `authoring schema`/`authoring compile`, `generate`, `new`, `validate`, `inspect`, `decompile`, `render`, `schema`, `types`, `describe`, and `ai generate`/`ai lab`.

## Architecture and roadmap source of truth

Run `cairn context` before broad repository exploration. `cairn.blueprint` owns
stable architecture responsibilities, accepted decisions live in
`meta/decisions/`, research in `meta/research/`, and work status in
`meta/todos/`. `ROADMAP.md` is the ordered view of those todos.

`dec.ai-authoring-layer` is binding: `SceneSpec` is the canonical lowered IR and
expert escape hatch, while complex AI generation targets the delivered
AuthoringSpec frontend. Keep complex generation skills aligned to the live
AuthoringSpec schema, authored stable IDs, source maps, semantic/runtime evidence,
and incremental operation seam. Raw SceneSpec guidance must remain bounded to
expert low-level work or unsupported typed concepts.

Before a PR that changes architecture or roadmap scope, update the relevant Cairn
artefacts and run:

```bash
cairn scan
cairn lint
```

## Architecture & Data Flow

The codebase is organized around typed authoring, explicit scene IR, binary generation and validation, plus a browser render path:

```
AuthoringSpec
  → serde → AuthoringSpec (src/authoring/)
  → deterministic lowering + source map
  → SceneSpec

JSON SceneSpec
  → serde → SceneSpec (src/builder/spec.rs)
  → build_scene() → Vec<Box<dyn RiveObject>>
  → encode_riv() → .riv file
  → validate_riv() / inspect_riv() (src/validator/mod.rs)
  → render() → embedded harness + Rive WASM + headless Chromium → PNG/manifest
```

**Key layers**:

1. **Authoring frontend** (`src/authoring/`): High-level typed visual, motion, behavior, stable-ID operation, deterministic lowering, and source-map boundary.
2. **Object Model** (`src/objects/`): Flat struct hierarchy implementing the `RiveObject` trait.
3. **Builder** (`src/builder/`): SceneSpec JSON deserialization, discovery metadata, validation, and object tree construction.
4. **Encoder** (`src/encoder/`): Serializes the flat object list into the `.riv` binary format.
5. **Validator** (`src/validator/`): Reads `.riv` bytes back for structural checks, inspection, and decompilation.
6. **Render** (`src/render/`): Serves the embedded harness and runtime assets, drives headless Chromium over CDP, captures PNGs, and computes previews.
7. **Discovery** (`src/discovery/`): Exposes SceneSpec schema, object types, valid parents, fields, and animatable-property metadata.
8. **Scaffold** (`src/scaffold/`): Emits known-good starter SceneSpec templates.
9. **AI Subsystem** (`src/ai/`): AuthoringSpec-first LLM provider path, incremental authored repair, evaluation suites, and built-in SceneSpec templates.
10. **MCP Server** (`src/mcp/`): Optional feature-gated Model Context Protocol server exposing AuthoringSpec and expert SceneSpec surfaces.

**Hierarchy model**: Objects are stored in a flat `Vec<Box<dyn RiveObject>>`. Parent-child relationships are tracked via `parent_id` (artboard-local index, 0-based, excluding Backboard). Each artboard has independent object/animation/interpolator index scopes.

## Key Directories

| Directory | Purpose |
|-----------|---------|
| `src/` | Main source code |
| `src/authoring/` | High-level AuthoringSpec schema, compiler, source maps, and stable-ID operations |
| `src/objects/` | Rive object type definitions and constants |
| `src/builder/` | SceneSpec JSON → scene object pipeline |
| `src/encoder/` | `.riv` binary format writer |
| `src/validator/` | Binary reader, parser, inspector, decompiler |
| `src/render/` | CDP browser rendering, PNG analysis, coverage previews |
| `src/discovery/` | SceneSpec schema, type, and animatable-property discovery |
| `src/scaffold/` | Known-good SceneSpec templates |
| `src/ai/` | AI-assisted AuthoringSpec generation, repair, and evaluation |
| `src/mcp/` | Optional MCP server (`mcp` feature) |
| `assets/` | Embedded Rive JavaScript/WASM runtime and render harness |
| `showcase/` | Ten authored SceneSpec gallery examples |
| `examples/authoring/` | Typed static, animated, interactive, and raw-escape AuthoringSpec examples |
| `skills/rive-animation/` | Bounded SceneSpec expert skill; complex AI generation routes through AuthoringSpec guidance |
| `tests/` | E2E CLI tests, JSON fixtures, Playwright runtime regression |
| `tests/fixtures/` | 66 JSON scene specs, including 3 intentionally invalid fixtures |
| `tests/playwright/` | Browser-based runtime and visual regression harness |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [George-RD/rive-rs-cli](https://github.com/George-RD/rive-rs-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
