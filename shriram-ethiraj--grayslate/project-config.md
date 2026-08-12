---
trigger: always_on
description: Reference for any AI coding agent (Claude Code, Codex, opencode, etc.) working on this repository.
---

# Grayslate — Agent Guidelines

Reference for any AI coding agent (Claude Code, Codex, opencode, etc.) working on this repository.

## Project Overview

**Grayslate** is a lightweight, cross-platform developer scratchpad. It features built-in functions for handling data formats like JSON and CSV, and allows users to add their own custom functions. Notes are auto-saved, can be custom-named by the user, and are automatically synced to Git.

## Tech Stack & Constraints

- **Desktop Framework:** Tauri v2
- **Backend Languages:** Rust
- **Frontend Framework:** Svelte 5 (SvelteKit with Static Adapter)
- **Frontend Language:** TypeScript
- **Editor Engine:** CodeMirror 6
- **Bundler:** Vite
- **Package Manager:** pnpm

---

## Where To Look

Keep this file compact. Detailed implementation notes belong in the skill files.

- Frontend patterns: `.agents/skills/svelte-frontend/SKILL.md`
- Code review playbook: `.agents/skills/code-review/SKILL.md`
- CodeMirror session model: `.agents/skills/codemirror-core/SKILL.md`
- Editor extension patterns: `.agents/skills/editor-extensions/SKILL.md`
- CSV table architecture: `.agents/skills/csv-architecture/SKILL.md`
- Naming and SQL naming architecture: `.agents/skills/naming-architecture/SKILL.md`
- Transformation architecture: `.agents/skills/transformations/SKILL.md`
- Sidebar search architecture: `.agents/skills/search-architecture/SKILL.md`
- Hotkeys: `.agents/skills/tanstack-hotkeys/SKILL.md`
- Memory reclamation: `.agents/skills/memory-management/SKILL.md`
- Tauri backend: `.agents/skills/tauri-backend/SKILL.md`
- Layout safety rules: `.agents/skills/layout-chain/SKILL.md`
- Typography & spacing: `.agents/skills/typography/SKILL.md`

Claude Code also sees these skills via `.claude/skills` (symlinked to `.agents/skills`) and can invoke them automatically or with `/skill-name`.

Language detection and naming logic live in workspace crates, not under `src-tauri/src/`: `crates/grayslate-langdetect/` and `crates/grayslate-langnaming/`. `src-tauri/src/detection.rs` and `src-tauri/src/naming.rs` are thin re-export shims — see the naming-architecture skill above for the full layout.

## Current High-Level Reality

- File open flows through `EditorWrapper.svelte` into Rust `read_file_content`, which strictly detects/decodes UTF-8, UTF-8 BOM, UTF-16 LE/BE, and confirmed Windows-1252 before returning UTF-8 IPC bytes. Raw and decoded input are capped at 200 MB.
- CodeMirror document state is preserved in a managed session even when the live editor view is destroyed.
- Find/replace uses a custom Svelte panel; CodeMirror still owns highlights and navigation on the main thread, while match-count/current-match stats are computed in Rust (`src-tauri/src/findstats.rs`, via `invoke("editor_find_scan", ...)`) so large-document scans don't block typing. There is no JS Web Worker in this flow.
- Built-in transformations use a shared Rust progress/cancellation context plus a chunked large-text transport; the frontend assembles chunked results into a CodeMirror `Text` rope and applies them as one undoable transaction.
- CSV table mode is Rust-backed and mounted on demand.
- CSV table edits mirror live into the preserved CodeMirror session only for sessions that start at or below 100,000 data rows; larger sessions skip live mirroring and return to text mode as a single undo step back to the pre-table state.
- Markdown preview uses Rust-side `pulldown-cmark` plus `ammonia`, with sanitized HTML returned over raw-byte IPC and custom scroll-sync hooks.
- Loader and memory-reclamation behavior are shared infrastructure, not CSV-specific logic.

---

## Architecture & Coding Standards

### 1. Frontend (Svelte 5 & TypeScript)

**> To know more about this topic, YOU MUST READ the `.agents/skills/svelte-frontend/SKILL.md` file.**

- **Embrace Svelte 5 Runes:** Exclusively use modern Svelte 5 signals (`$state`, `$derived`, `$effect`, `$props`). Avoid Svelte 4 legacy features.
- **Strong Typing:** Do not use `any`. Use strict TypeScript interfaces.
- **Vite Native:** Let Vite handle assets and bundling.
- **Memory Efficiency:** Aggressively clean up memory on component unmount (`onDestroy`). Explicitly nullify `$state` variables, large arrays, objects, and external DOM references (e.g., CodeMirror views) when a component is destroyed, especially in expensive views (Diff, CSV, Markdown).

### 2. Editor Integration (CodeMirror 6)

**> To know more about core integration, YOU MUST READ the `.agents/skills/codemirror-core/SKILL.md` file.**
**> To know more about custom extensions, YOU MUST READ the `.agents/skills/editor-extensions/SKILL.md` file.**

- Keep `EditorState` separate from Svelte's `$state` to avoid reactivity loops.
- Perform updates via `Transaction`s.
- Preserve document state in a managed session even when the live `EditorView` is unmounted.
- Use compartments for language/theme/word-wrap reconfiguration instead of rebuilding the editor state unnecessarily.
- **Performance:** Cap Lezer tree traversals to avoid freezing the main thread.

### 3. Desktop / Backend (Tauri v2 & Rust)

**> To know more about backend rules, YOU MUST READ the `.agents/skills/tauri-backend/SKILL.md` file.**

- Ensure usage of Tauri **v2** APIs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shriram-ethiraj/grayslate](https://github.com/shriram-ethiraj/grayslate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
