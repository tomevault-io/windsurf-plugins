---
trigger: always_on
description: > **This is a Cursor product.** All AI features are powered by Cursor's backend—no local models, no API keys, no provider configuration.
---

# Formula: Agent Guidelines

> **This is a Cursor product.** All AI features are powered by Cursor's backend—no local models, no API keys, no provider configuration.

---

## ⛔ READ THIS FIRST ⛔

**Before doing ANYTHING, understand these rules. They are non-negotiable.**

1. **Memory limits are critical** — see [Development Environment](#development-environment)
2. **Use wrapper scripts for cargo** — `bash scripts/cargo_agent.sh`, never bare `cargo`
3. **Excel compatibility is non-negotiable** — 100% formula behavioral compatibility
4. **Cursor product identity** — all AI goes through Cursor servers, period

---

## Workstream Instructions

**You are assigned to ONE workstream. Read your workstream file and follow it.**

| Workstream | File | Focus |
|------------|------|-------|
| **A: Core Engine** | [`instructions/core-engine.md`](./instructions/core-engine.md) | Formula parser, dependency graph, functions, Rust/WASM |
| **B: UI/UX** | [`instructions/ui.md`](./instructions/ui.md) | Canvas grid, formula bar, panels, React/TypeScript |
| **C: File I/O** | [`instructions/file-io.md`](./instructions/file-io.md) | XLSX/XLSB reader/writer, round-trip preservation |
| **D: AI Integration** | [`instructions/ai.md`](./instructions/ai.md) | Context management, tool calling, Cursor backend |
| **E: Collaboration** | [`instructions/collaboration.md`](./instructions/collaboration.md) | CRDT sync, presence, version history, Yjs |
| **F: Platform** | [`instructions/platform.md`](./instructions/platform.md) | Tauri shell, native integration, distribution |

**All workstream files reference back to this document.** This is the shared foundation.

---

## Mission Statement

Build a spreadsheet that achieves **100% Excel compatibility** while introducing **radical AI-native capabilities** and **modern architectural foundations**.

**Strategic Imperatives:**

1. **Zero-Compromise Excel Compatibility** — every `.xlsx` loads perfectly
2. **AI as Co-Pilot, Not Gimmick** — woven into formulas, not a chatbot sidebar
3. **Performance That Scales** — 60fps with millions of rows
4. **Modern Foundation** — Git-like versioning, real-time collaboration, Python/TypeScript scripting
5. **Win Power Users First** — finance modelers, data analysts, quant researchers

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  PRESENTATION LAYER (TypeScript/React)                                       │
│  ├── Canvas-Based Grid Renderer (60fps virtualized scrolling)               │
│  ├── Formula Bar with AI Autocomplete                                        │
│  ├── Command Palette (Cmd/Ctrl+Shift+P)                                      │
│  └── AI Chat Panel                                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│  CORE ENGINE (Rust/WASM - runs in Worker thread)                            │
│  ├── Formula Parser (A1/R1C1/Structured References)                         │
│  ├── Dependency Graph (incremental dirty marking)                           │
│  ├── Calculation Engine (multi-threaded, SIMD)                              │
│  └── Function Library (500+ Excel-compatible)                               │
├─────────────────────────────────────────────────────────────────────────────┤
│  DATA LAYER                                                                  │
│  ├── SQLite (CRDT-enabled for sync)                                         │
│  ├── Format Converters (XLSX, XLSB, CSV, Parquet)                           │
│  └── Yjs (CRDT for collaboration)                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│  AI LAYER (Cursor-managed)                                                   │
│  ├── Cursor AI Backend (all inference)                                      │
│  ├── Context Manager (schema extraction, sampling, RAG)                     │
│  └── Tool Calling Interface                                                 │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Key Decisions:**

| Component | Choice | Rationale |
|-----------|--------|-----------|
| Desktop Framework | Tauri + Rust | 10x smaller than Electron, memory safety |
| Calculation Engine | Rust → WASM | Near-native performance in browser |
| Grid Rendering | Canvas-based | Only way to hit 60fps with millions of rows |
| Collaboration | CRDT (Yjs) | Better offline support than OT |
| AI | Cursor Backend | Cursor controls all AI inference |

---

## Development Environment

### ⚠️ Memory Management (CRITICAL)

This machine runs **~200 concurrent agents**. Memory is the constraint.

### Local-only agent files (`scratchpad.md`, `handoff.md`)

Some agent harnesses create `scratchpad.md` (working notes) and `handoff.md` (planner handoff) in
the repo root. These files are **gitignored** and should **never** be committed.

If they become hard to read because diff/patch text was pasted into them, clean them up locally by
converting any accidental literal backslash-`n` fragments back into real newlines / normal Markdown

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wilson-anysphere/formula](https://github.com/wilson-anysphere/formula) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
