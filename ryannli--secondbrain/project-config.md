---
trigger: always_on
description: This is the canonical agent instruction file for this repository.
---

# Second Brain - Agent Guidelines

This is the canonical agent instruction file for this repository.
`CLAUDE.md` should be a symlink to this file so Claude Code and Codex share the same project context.

> A persistent cognitive workspace that manages information flow for knowledge workers.

## Core Philosophy

**The Problem**: AI content grows exponentially, but human attention scales linearly.

**Our Solution**: AI handles information triage and organization; humans focus on judgment and decisions.

**Key Principles**:
1. **Local-first**: Filesystem = single source of truth. Web UI is just a view.
2. **User ownership**: All AI outputs are editable. This is the user's second brain.
3. **Security boundary**: All operations stay within user-specified root directory.
4. **Structure is optional**: System doesn't force organization; respects user-created structure.
5. **First principles, no hardcoding**: Solve problems from first principles. Avoid hardcoded rules or brittle parsing logic. When dealing with messy/unstructured data (HTML, PDFs, etc.), prefer letting the AI model interpret the content rather than writing fragile extraction code. Ensure generalizability over edge-case handling.
6. **Minimal code**: Less code is always better. **Knowing when to delete code is more important than knowing how to write it.** Don't write unnecessary code. If something can be achieved with fewer lines, do it. Actively look for code to delete - dead code, redundant logic, over-abstractions. The best code is no code.
7. **Think globally, not incrementally**: Don't just patch the immediate problem. Step back and ask: Should this be refactored? Are there similar patterns to unify? Incremental fixes accumulate into debt.

---

## Information Processing Model

### Two Modes of Analysis

| Mode | Purpose | Outputs |
|------|---------|---------|
| **Digestion** | Help user understand content | Summary, key concepts, structure, background |
| **Critique** | Help user evaluate content | Hidden assumptions, issues, verification needs |

### Claim-Level Granularity

A "claim" = a statement that can be independently evaluated for truth/agreement.
- Extract claims from each source
- Compare across sources (find similar, contradicting)
- MVP: brute-force comparison; future: vector search

### Triage Card

Every source gets a quick assessment:
- Read time estimate
- Information density score
- Novelty score (vs user's existing knowledge)
- One-line recommendation (deep read / skim / skip)

---

## File System Architecture

```
{root}/
├── config.json                 # User preferences (theme, settings)
├── library/
│   ├── {folder}/               # User-created folders (optional)
│   │   └── {id}/
│   │       ├── meta.json       # IMMUTABLE: metadata captured at creation
│   │       ├── original.html   # IMMUTABLE: raw HTML from capture
│   │       ├── content.md      # DERIVED: processed content (editable)
│   │       ├── analysis.json   # DERIVED: AI analysis (editable)
│   │       ├── README.md       # DERIVED: human-readable Triage Card
│   │       └── error.txt       # Only present if processing failed
│   └── {id}/                   # Sources can also be at top level
├── .index/                     # Global indices (claims, graph)
└── .cache/
```

### Organization Rules
- **System default**: New content goes to `library/` top level
- **User freedom**: Users can create folders, move sources freely
- **Detection**: Any directory containing `meta.json` = a source entry
- **UI options**: Tree view (show hierarchy) or Flat view (ignore hierarchy)

### meta.json (Immutable)

Written once at capture time, **never modified**:
```json
{
  "id": "abc123",
  "source_url": "https://...",
  "created_at": "2024-01-28T...",
  "type": "html",
  "original_file": "original.html",
  "original_title": "Page title at capture time"
}
```

### content.md (Derived)

Processed content with editable title in frontmatter:
```markdown
---
title: "AI-improved or user-edited title"
---

[Processed content here - markdown formatted]
```

### Processing Status (File-based)

Status is **inferred from file existence**, not stored:

| State | Files Present |
|-------|---------------|
| Processing | `meta.json` only |
| Ready | `meta.json` + `content.md` + `analysis.json` |
| Failed | `meta.json` + `error.txt` |

### Data Model: Original vs Derived

**CRITICAL**: Understand which data is original (immutable after capture) vs derived (can be regenerated).

| File | Type | Description |
|------|------|-------------|
| `meta.json` | **ORIGINAL** | Metadata at capture time. **NEVER modify.** |
| `original.html` / `original.txt` | **ORIGINAL** | Raw content from first capture. **NEVER overwrite.** |
| `content.md` | **DERIVED** | Processed/extracted content. Can regenerate. |
| `analysis.json` | **DERIVED** | AI analysis. Can regenerate. |
| `README.md` (triage card) | **DERIVED** | Human-readable summary. Can regenerate. |

**Reanalyze operation**:
1. Read metadata from `meta.json`
2. Read raw content from `original.html` / `original.txt`
3. Re-extract content → update `content.md`
4. Re-run AI analysis → update `analysis.json`
5. **NEVER** modify `meta.json` or `original.html`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ryannli/secondbrain](https://github.com/ryannli/secondbrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
