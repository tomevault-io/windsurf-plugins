---
trigger: always_on
description: Use when: searching across multiple files, investigating cross-cutting features, running parallel tests, complex research.
---

# Eigenpal DOCX Editor

## Project Context

Bun + React (TSX) WYSIWYG editor for DOCX files:

1. **Display DOCX** — render with full WYSIWYG fidelity per ECMA-376 spec
2. **Insert docxtemplater variables** — `{variable}` mappings with live preview

Two entry points: `src/index.ts` (full UI), `src/headless.ts` (Node.js API).
Client-side only. No backend.

---

## Verify Commands

```bash
# Fast cycle (use this 95% of the time)
bun run typecheck && npx playwright test --grep "<pattern>" --timeout=30000 --workers=4

# Single test file
bun run typecheck && npx playwright test tests/formatting.spec.ts --timeout=30000

# Only affected test files (use this after targeted changes)
bun run typecheck && npx playwright test tests/formatting.spec.ts tests/demo-docx.spec.ts --timeout=30000 --workers=4

# Full suite (only for final validation — NEVER run casually, 500+ tests)
bun run typecheck && npx playwright test --timeout=60000 --workers=4
```

### Test File Mapping

| Feature Area          | Test File                      | Quick Verify Pattern        |
| --------------------- | ------------------------------ | --------------------------- |
| Bold/Italic/Underline | `formatting.spec.ts`           | `--grep "apply bold"`       |
| Alignment             | `alignment.spec.ts`            | `--grep "align text"`       |
| Lists                 | `lists.spec.ts`                | `--grep "bullet list"`      |
| Colors                | `colors.spec.ts`               | `--grep "text color"`       |
| Fonts                 | `fonts.spec.ts`                | `--grep "font family"`      |
| Enter/Paragraphs      | `text-editing.spec.ts`         | `--grep "Enter"`            |
| Undo/Redo             | `scenario-driven.spec.ts`      | `--grep "undo"`             |
| Line spacing          | `line-spacing.spec.ts`         | `--grep "line spacing"`     |
| Paragraph styles      | `paragraph-styles.spec.ts`     | `--grep "Heading"`          |
| Toolbar state         | `toolbar-state.spec.ts`        | `--grep "toolbar"`          |
| Cursor-only ops       | `cursor-paragraph-ops.spec.ts` | `--grep "cursor only"`      |
| Comments sidebar      | `comments-sidebar.spec.ts`     | `--grep "Comments sidebar"` |

**When touching anything in these paths, run `comments-sidebar.spec.ts`:**

- `packages/react/src/components/UnifiedSidebar.tsx`
- `packages/react/src/components/sidebar/**`
- `packages/react/src/hooks/useCommentSidebarItems.tsx`
- `packages/react/src/paged-editor/PagedEditor.tsx` → `updateSelectionOverlay` / `onSelectionChange`
- `packages/react/src/components/DocxEditor.tsx` → `onSelectionChange` handler, `expandedSidebarItem` state

**Known flaky tests:** `formatting.spec.ts` (bold toggle/undo/redo), `text-editing.spec.ts` (clipboard ops).

### Avoid Hanging

- **Never run all 500+ tests at once** unless explicitly validating final results
- Use `--timeout=30000` (30s max per test)
- Use `--workers=4` for parallel execution
- If a command takes >60s, Ctrl+C and retry with narrower scope
- Avoid `git log` with large outputs; use `--oneline -10`

---

## Subagents — Use For Complex Tasks

Spin up subagents for parallel work using the Task tool:

- **Explore agent** — codebase exploration, finding files, understanding architecture
- **Plan agent** — designing implementation approaches
- **Bash agent** — running commands, git operations

Use when: searching across multiple files, investigating cross-cutting features, running parallel tests, complex research.

---

## ECMA-376 Reference

```bash
reference/quick-ref/wordprocessingml.md   # Paragraphs, runs, formatting
reference/quick-ref/themes-colors.md      # Theme colors, fonts, tints
reference/ecma-376/part1/schemas/wml.xsd  # WordprocessingML schema
reference/ecma-376/part1/schemas/dml-main.xsd # DrawingML schema
```

---

## WYSIWYG Fidelity — Hard Rule

Output must look identical to Microsoft Word. Must preserve: fonts, theme colors, styles, character formatting, tables (borders, shading, merged cells), headers/footers, section layout (margins, page size, orientation).

---

## Editor Architecture — Dual Rendering System

**This editor has TWO separate rendering systems. You MUST understand which one you're working with.**

```
┌──────────────────────────────────────────────────────────────┐
│  HIDDEN ProseMirror (left: -9999px)                          │
│  - Real editing state (selection, undo/redo, commands)       │
│  - Receives keyboard input                                   │
│  - CSS class: .paged-editor__hidden-pm                       │
│  - Component: src/paged-editor/HiddenProseMirror.tsx         │
└──────────────────────────────────────────────────────────────┘
        │ state changes trigger re-render ↓
┌──────────────────────────────────────────────────────────────┐
│  VISIBLE Pages (layout-painter)                              │
│  - What the user actually sees                               │
│  - Static DOM, re-built from PM state on every change        │
│  - Has its own rendering logic (NOT toDOM)                   │
│  - CSS class: .paged-editor__pages                           │
│  - Entry: src/layout-painter/renderPage.ts                   │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eigenpal/docx-editor](https://github.com/eigenpal/docx-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
