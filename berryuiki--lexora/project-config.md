---
trigger: always_on
description: This document provides context, architectural constraints, project conventions, and operational instructions for AI coding agents and autonomous assistants contributing to the **Lexora** codebase.
---

# AGENTS.md — AI Agent & Automation Guidelines

This document provides context, architectural constraints, project conventions, and operational instructions for AI coding agents and autonomous assistants contributing to the **Lexora** codebase.

---

## 🧠 Architectural Mental Model

Lexora is a **local-first, in-place WYSIWYG Markdown reader & editor** built on **Tauri 2 + Rust + SolidJS**.

```
┌────────────────────────────────────────────────────────┐
│               Frontend (SolidJS + Webview)             │
│   • Reactive UI Components (Sidebar, TOC, Viewport)    │
│   • Editor Engine (Milkdown / ProseMirror)             │
│   • Theme & Settings Signals (localStorage / DOM)     │
│   • Typed IPC Wrappers (invoke / listen)               │
└──────────────────────────┬─────────────────────────────┘
                           │ IPC Bridge (JSON / Events)
┌──────────────────────────▼─────────────────────────────┐
│                 Backend (Rust Native App)              │
│   • State Management (Mutex<AppState>)                 │
│   • File I/O & Atomic Writes (tempfile -> rename)      │
│   • Markdown Parsing (pulldown-cmark zero-copy AST)   │
│   • Code Syntax Highlighting (syntect)                 │
│   • File System Watcher (notify crate)                 │
└────────────────────────────────────────────────────────┘
```

---

## ⛔ Core Architectural Constraints

When implementing features or refactoring:

1. **Strict IPC Boundary**:
   - **NEVER** attempt direct filesystem access or heavy computation in the frontend.
   - All file reading, writing, renaming, directory listing, AST parsing, and code highlighting **MUST** reside in the Rust backend under `src-tauri/src/services/` and be exposed via `src-tauri/src/commands/`.
2. **Atomic File Operations**:
   - Never overwrite document files directly in place. Always use the atomic write pattern (`write to .tmp` -> `flush` -> `fs::rename`) implemented in `fs_service.rs` to guarantee crash resilience.
3. **Tauri v2 Least-Privilege Capabilities**:
   - Permissions are defined strictly in `src-tauri/capabilities/default.json`. Do not introduce wildcard permissions (`fs:allow-all` or `shell:allow-all`). Scope permissions precisely.
4. **Pure Markdown Source of Truth**:
   - Lexora's on-disk and in-memory representation is pure Markdown. Avoid introducing proprietary metadata, custom frontmatter, or incompatible AST wrappers that could break roundtripping.
5. **DOM Performance Budget**:
   - Keystroke latency must remain under 16ms (60 FPS).
   - Use SolidJS fine-grained reactivity (`createSignal`, `createMemo`, `<Show>`, `<For>`) rather than recreating entire DOM trees.

---

## 📁 Repository Directory Structure

```
Lexora/
├── .github/                      # GitHub issue & PR templates
├── docs/                         # Project documentation suite
│   ├── ARCHITECTURE.md           # System design & data flow
│   ├── COLLABORATION.md          # Team workflow & review rules
│   ├── DESIGN_DECISIONS.md       # ADRs (SolidJS, Milkdown, etc.)
│   ├── DEVELOPMENT.md            # Developer setup & debug guide
│   ├── MILESTONES.md             # Project milestones & schedule
│   ├── NEXT_STEPS.md             # Phase 2 implementation blueprint
│   └── ROADMAP.md                # Phased feature roadmap & MoSCoW
├── src/                          # SolidJS Frontend
│   ├── components/               # UI components
│   │   ├── Editor/               # Milkdown editor integration
│   │   ├── MarkdownView/         # Rendered Markdown viewer
│   │   ├── Sidebar/              # TOC sidebar & File tree
│   │   └── StatusBar/            # Bottom metadata status bar
│   ├── lib/
│   │   └── tauri/                # Typed IPC wrappers (commands.ts, events.ts)
│   ├── store/                    # SolidJS reactive state signals
│   ├── styles/                   # Tailwind CSS 4 & theme variables
│   └── types/                    # Shared TypeScript interfaces
├── src-tauri/                    # Rust Backend (Tauri 2)
│   ├── capabilities/             # Tauri v2 security scopes
│   ├── icons/                    # Multi-platform app icons
│   ├── src/
│   │   ├── commands/             # #[tauri::command] handlers
│   │   ├── models/               # Serde data structs
│   │   ├── services/             # Core business logic (parser, watcher, fs)
│   │   ├── lib.rs                # App builder, plugin init, command routing
│   │   ├── main.rs               # Windows entry point
│   │   └── state.rs              # Managed AppState
│   ├── Cargo.toml                # Rust dependencies & metadata
│   └── tauri.conf.json           # Tauri window & bundle config
├── AGENTS.md                     # This agent guideline document
├── CHANGELOG.md                  # Keep a Changelog format
├── package.json                  # Frontend dependencies & scripts
└── tsconfig.json                 # TypeScript compiler configuration
```

---

## 🛠️ Verification & Build Commands

Before committing or concluding a task, run the following verification commands:

```bash
# 1. Run all Rust unit tests
cargo test --manifest-path src-tauri/Cargo.toml


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BerryUIKI/Lexora](https://github.com/BerryUIKI/Lexora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
