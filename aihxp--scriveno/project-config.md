---
trigger: always_on
description: Scriveno is a spec-driven writing, publishing, and translation pipeline that runs inside AI coding agents (Codex, Cursor, Gemini CLI). It covers the full lifecycle from blank page to publication-ready manuscript or technical document set -- including voice profiling, adaptive work types, autonomous drafting, illustration, translation, and multi-format export. Supports 50 work types with tradition-native vocabulary (novels use chapters, screenplays use acts, runbooks use procedures, Quran comment
---

## Project

**Scriveno**

Scriveno is a spec-driven writing, publishing, and translation pipeline that runs inside AI coding agents (Codex, Cursor, Gemini CLI). It covers the full lifecycle from blank page to publication-ready manuscript or technical document set -- including voice profiling, adaptive work types, autonomous drafting, illustration, translation, and multi-format export. Supports 50 work types with tradition-native vocabulary (novels use chapters, screenplays use acts, runbooks use procedures, Quran commentaries use surahs).

**Core Value:** **Drafted prose sounds like the writer, not like AI.** The Voice DNA system profiles the writer across 15+ dimensions and loads that profile into every drafter agent invocation. If voice fidelity breaks, trust breaks, and no other feature matters.

### Constraints

- **Architecture**: Must remain a pure skill/command system -- no compiled code, no runtime dependencies beyond Node.js for the installer
- **Voice fidelity**: Every feature must preserve the Voice DNA pipeline -- fresh context per atomic unit, STYLE-GUIDE.md loaded first
- **Backward compatibility**: Existing commands and templates must continue working as new features are added
- **Plan authority**: If a command file contradicts the product plan, fix the command -- plan is canonical (section 15 for command specs)
- **Progressive disclosure**: Onboarding asks 3 questions max; depth is optional and additive
- **Runtime credibility**: `>=20.0.0` is the installer compatibility floor. For new installs, prefer a currently supported LTS such as Node.js 24. `docs/runtime-support.md` is the canonical runtime matrix, and installer targets are not interchangeable proof of host-runtime parity.
## Technology Stack

## Architecture Constraint
- Export tools are **external CLI binaries** the agent invokes via shell, not npm dependencies
- The agent generates intermediate files (markdown, HTML, Typst) then calls converters
- Scriveno's `package.json` stays dependency-free; tools are prerequisites the user installs
- The installer (`bin/install.js`) should detect and guide installation of prerequisites
## Recommended Stack
### Document Conversion Engine
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| **Pandoc** | Current stable 3.x | Universal document converter | De facto standard for markdown-to-anything. Handles EPUB, DOCX, PDF, LaTeX, Typst, HTML. One tool covers most export needs and has a large ecosystem of filters and templates. | HIGH |
### PDF Generation
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| **Typst** | Current stable | PDF engine for Pandoc | Clean syntax, smaller install footprint than TeX Live, and native Pandoc support through `--pdf-engine=typst`. Use for general book interiors; reserve LaTeX for academic class requirements. | HIGH |
| **XeLaTeX** (fallback) | TeX Live 2025 | Academic/math-heavy PDF | Only needed if Typst cannot handle specialized mathematical notation or journal-specific LaTeX templates. Most creative writing does not need this. | MEDIUM |
### EPUB Generation
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| **Pandoc** (built-in) | 3.9.x | EPUB 3 generation | Pandoc's EPUB output is production-quality. Supports custom CSS, metadata, cover images, table of contents. Used by published authors and small presses. No additional tool needed. | HIGH |
### DOCX Generation
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| **Pandoc** (built-in) | 3.9.x | Manuscript DOCX and formatted DOCX | Supports reference documents (`.docx` templates) for both standard manuscript format (12pt Courier, double-spaced) and formatted/designed output. | HIGH |
### Screenplay Formats (Fountain + FDX)
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| **Afterwriting CLI** | 1.8.x | Fountain to PDF | Node.js-based, npm-installable (`npm i -g afterwriting`). Generates industry-standard screenplay PDFs with page numbers, scene headers, proper formatting. Also provides screenplay statistics. | MEDIUM |
| **Screenplain** | 0.9.x | Fountain to FDX + HTML | Python-based (`pip install screenplain`). Only reliable open-source Fountain-to-FDX converter. FDX is Final Draft's XML format -- essential for screenplay submission. | MEDIUM |
### LaTeX Output
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aihxp/scriveno](https://github.com/aihxp/scriveno) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
