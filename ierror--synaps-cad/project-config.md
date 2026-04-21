---
trigger: always_on
description: This document describes the SynapsCAD architecture for AI agents (LLMs, copilots, and automated tools) that work with this codebase. It covers the system design, plugin structure, and the **part labeling** concept that enables users to reference specific 3D geometry parts in conversations with AI.
---

# AGENTS.md — Architecture Overview for AI Agents

## Purpose

This document describes the SynapsCAD architecture for AI agents (LLMs, copilots, and automated tools) that work with this codebase. It covers the system design, plugin structure, and the **part labeling** concept that enables users to reference specific 3D geometry parts in conversations with AI.

## System Architecture

SynapsCAD is a single-binary Rust desktop app built on **Bevy 0.15** (ECS game engine) with **egui** for UI.

### Data Flow

```
SynapsCAD code (editor)
    ↓  trigger_compilation_system
Compiler thread (scad-rs parser → AST evaluator → csgrs CSG → boolmesh)
    ↓  mpsc channel
poll_compilation_system
    ↓  spawns Bevy entities
3D Viewport (Bevy renderer)
    ↓  egui overlay
Part Labels (@1, @2, ...)
    ↓  system prompt injection
AI Chat (genai → Anthropic/OpenAI/Gemini/...)
    ↓  code block extraction
Code Editor (auto-apply)
```

### Plugin Structure (`src/plugins/`)

| Plugin              | File             | Responsibility                                                           |
| ------------------- | ---------------- | ------------------------------------------------------------------------ |
| `ScenePlugin`       | `scene.rs`       | Camera, lights, axes, grid setup                                         |
| `CodeEditorPlugin`  | `code_editor.rs` | OpenSCAD text editor, undo/redo, view detection (`$view` variable)       |
| `CompilationPlugin` | `compilation.rs` | Triggers compilation, spawns mesh entities with `CadModel` + `PartLabel` |
| `CameraPlugin`      | `camera.rs`      | Orbit/pan/zoom controls, zoom-to-fit, keyboard toggles (G=gizmos, L=labels) |
| `UiPlugin`          | `ui.rs`          | egui side panel layout, viewport toolbar, label overlays                 |
| `AiChatPlugin`      | `ai_chat.rs`     | AI streaming chat with context injection                                 |
| `PersistencePlugin` | `persistence.rs` | Save/load settings and code                                              |

### Key Files

| File                | Purpose                                                                                                                     |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `src/compiler/`     | OpenSCAD → triangle mesh pipeline. Modular directory handling evaluation, geometry, rendering, and types.                    |
| `src/main.rs`       | App entry point, registers all plugins                                                                                      |
| `src/app_config.rs` | Developer constants (not user-facing)                                                                                       |

## Labels (`@N`)

### Part Numbering

Parts use a `@N` numbering scheme:

- Parts are numbered `@1`, `@2`, ... (1-based, in order of top-level geometry)

Each part gets a **unique color** (from `PART_PALETTE` or from `color()` in code). The label is rendered as a billboard overlay at the part's AABB center.

### Why This Matters for Agents

When a user says _"make @2 taller"_ or _"change the color of @1"_, the AI agent knows exactly which geometric part is being referenced. The label system provides:

1. **Visual identification** — colored labels in the viewport
2. **AI context** — part index, color, and bounding box injected into the system prompt
3. **Stable references** — part numbers correspond to top-level geometry statements in order

### How Parts Map to Code

Parts are created by the compiler in the order of top-level geometry statements:

```openscad
cube(10);           // → @1
translate([20,0,0])
    sphere(5);      // → @2
```

## For Agent Developers

### Code Block Namespace

AI-generated code uses the **`synapscad`** namespace in fenced code blocks:

- ` ```synapscad ` — wraps the full script to replace in the editor

The parser in `extract_openscad_code()` (`ai_chat.rs`) extracts code from the block and replaces the entire editor buffer.

### Verification & Error Recovery

The AI chat system includes automatic verification and error recovery loops:

1. **Verification Loop**: After AI produces code, the system waits for compilation, then sends a verification prompt with fresh renders for the AI to confirm correctness.
2. **Error Recovery**: If AI-generated code causes a compilation error (parse error, syntax error, etc.), the error is automatically sent back to the AI with a request to fix it. The AI then produces corrected code and the cycle continues.

The `VerificationState` enum (`ai_chat.rs`) tracks this flow:
- `Idle` — no active verification
- `WaitingForCompilation` — AI produced code, waiting for compile to finish
- `ReadyToVerify` — compilation succeeded, ready to send verification prompt
- `Verifying(n)` — currently in verification round N
- `ErrorRecovery(err)` — compilation failed, send error back to AI


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ierror/synaps-cad](https://github.com/ierror/synaps-cad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
