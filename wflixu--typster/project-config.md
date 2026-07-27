---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Typster is a modern cross-platform Typst editor built with Tauri 2.0 (Rust backend) and Vue 3 (frontend), featuring a Typst-like WYSIWYG editing experience. The application provides seamless "what you see is what you get" document authoring with real-time mathematical formula rendering, similar to Typora, but specifically designed for Typst markup language.

## Architecture

### 1. **Overall Architecture**

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Backend       │    │   Configuration │
│   (Vue 3)        │    │   (Rust)        │    │   System        │
├─────────────────┤    ├─────────────────┤    ├─────────────────┤
│ • WYSIWYG Editor│    │ • Typst Service │    │ • TOML-based    │
│ • Tiptap Core   │    │ • IPC Commands   │    │ • Global/Project│
│ • Math Rendering│    │ • File System    │    │ • Runtime       │
│ • Formula Nodes │    │ • Compiler       │    │ • Validation    │
│ • State Mgmt    │    │ • Package Mgmt  │    │ • Repair        │
└─────────────────┘    └─────────────────┘    └─────────────────┘
        │                       │                       │
        └───────────────────────┼───────────────────────┘
                                │
                    ┌─────────────────┐
                    │   IPC Bridge    │
                    │ (Tauri API)     │
                    └─────────────────┘
```

### 2. **Technology Stack**

**Frontend (Vue 3 + TypeScript)**
- Vue 3 with Composition API
- TypeScript for type safety
- PrimeVue UI library
- Tiptap for WYSIWYG editing (primary editor)
- Custom math formula nodes
- KaTeX/MathJax for math rendering
- Pinia for state management
- Vue Router for navigation

**Backend (Rust + Tauri 2.0)**
- Rust 2021 edition
- Tauri 2.0 for cross-platform native apps
- Complete Typst 0.14.0 ecosystem
- TOML configuration
- Tokio for async runtime
- Serde for serialization

**Key Dependencies**
- **Frontend**: `@tiptap/vue-3`, `@tiptap/pm`, `katex`, `primevue`, `pinia`
- **Backend**: `typst = "0.14.0"` (complete ecosystem), `tauri = "2.9.3"`, `tokio`, `serde`, `toml`

## Development Commands

### **Development Environment Setup**

```bash
# Install frontend dependencies
pnpm install

# Run development server (Tauri + Vite)
pnpm start    # or `pnpm tauri dev`
# Runs on http://localhost:1420

# Build for production
pnpm pack     # or `pnpm tauri build`
# Creates platform-specific binaries
```

### **Available Scripts**

```json
{
  "dev": "vite",                    # Frontend dev server
  "build": "vue-tsc --noEmit && vite build",
  "preview": "vite preview",       # Preview production build
  "tauri": "tauri",                # Tauri CLI wrapper
  "td": "tauri dev",                # Tauri development mode
  "tb": "tauri build",              # Tauri production build
  "start": "pnpm tauri dev",       # Start full development
  "pack": "pnpm tauri build"        # Production packaging
}
```

### **Development Workflow**

1. **Frontend Development**: `pnpm dev` - Hot reload Vue components
2. **Full Development**: `pnpm start` - Tauri app with devtools
3. **Production Build**: `pnpm pack` - Platform-specific binaries

## Critical Technical Patterns

### 1. **IPC Command System**

The application uses Tauri's command system for frontend-backend communication:

```rust
// src-tauri/src/lib.rs - Command Registration
.invoke_handler(tauri::generate_handler![
    cmds::doc::load_doc_from_path,
    cmds::fs::fs_list_dir,
    cmds::fs::fs_read_file_text,
    cmds::typst::typst_compile_doc,
    cmds::typst::typst_render,
    cmds::typst::typst_autocomplete,
    cmds::typst::export_pdf,
])
```

**Key Commands:**
- `typst_compile_doc` - Compile Typst documents
- `fs_read_file_text` - File system operations
- `export_pdf` - PDF export functionality

### 2. **WYSIWYG Editor Architecture**

The core is a unified Typst-like editor using Tiptap as the foundation:

```
TypstEditor.vue (Main Editor)
├── TiptapEditor.vue
│   ├── Rich text editing foundation
│   ├── Typst-specific extensions
│   ├── Math formula nodes
│   ├── Markdown-like shortcuts
│   └── Real-time rendering
├── MathNode.ts (Custom ProseMirror Node)
│   ├── Click-to-edit behavior
│   ├── LaTeX math rendering
│   ├── Inline and display modes
│   └── Blur-to-render functionality
└── TypstExtensions.ts
    ├── Typst markup support
    ├── Heading shortcuts (# ##)
    ├── Bold/italic shortcuts
    └── Auto-completion
```

**Math Formula Editing Flow:**
```typescript
// Click-to-edit, blur-to-render pattern
const handleMathNodeClick = (mathNode: MathNode) => {
  // Switch to edit mode
  mathNode.setEditable(true)
  // Focus the math input
  mathNode.focusEditor()
}

const handleMathNodeBlur = (mathNode: MathNode) => {
  // Render back to visual mode
  mathNode.setEditable(false)
  // Convert LaTeX to rendered math
  renderMathFormula(mathNode)
}
```

### 3. **Typst Compilation Pipeline**

The backend implements a complete Typst 0.14.0 compilation pipeline:

```rust
// src-tauri/src/typst_service/compiler.rs
pub fn compile(timer: &mut Timer, args: &CompileArgs) -> StrResult<()> {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wflixu/typster](https://github.com/wflixu/typster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
