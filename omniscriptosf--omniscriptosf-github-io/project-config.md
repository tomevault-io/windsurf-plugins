---
trigger: always_on
description: <!-- Master rules for human and AI collaborators -->
---

<!-- /AGENTS.md -->
<!-- Master rules for human and AI collaborators -->
<!-- Why: keep work simple, fast, and safe -->
<!-- RELEVANT FILES: README.md, CONTRIBUTING.md, PHASE_2_PROGRESS.md -->

# AGENTS Master Guide

This is the source of truth for how humans and AI work together.

## GOAL
Ship value fast. Write clean, simple, modular code. Do exactly what is asked.

## ABOUT THE PRODUCT
OmniScript Format: universal document DSL for developers and AI agents.

Resources are limited. Favor speed and simplicity. Pick the 80:20 solution.

## PROJECT HISTORY AND VISION

**Origin**: October 2025. Created to unify documents, presentations, spreadsheets into one Git-friendly, AI-friendly plain-text format.

**Problem**: Binary formats (DOCX, PPTX, XLSX) hostile to version control and AI.

**Solution**: Write once in .osf, export to everything.

**Version History**:
- v0.5.0 (Oct 2025): First public release
- v0.6.0 (Oct 2025): Phase 1 complete, 3 npm packages, 79 tests
- v0.1.0 (Oct 2025): VSCode extension
- v1.0.0 (Target Q4 2025): Charts, diagrams, code blocks

**Current**: Phase 2 (40% done). Building ecosystem: VSCode extension, examples, docs.

**Vision 2026**: Reference implementation for universal document DSLs. Real-time collaboration, visual editor, plugin system.

## MODUS OPERANDI
Prefer boring solutions. Cut scope early. Avoid hidden complexity. Explain what and why.

## TECH STACK
Backend: Node.js 22, TypeScript 5.8. Frontend: Next.js 15.3, React 18, Tailwind 3.4. Infra: Docker, GitHub Actions, pnpm 10.12. Data: File-based, no database.

## ARCHITECTURE

**System Flow**:
```
.osf file → Parser → AST → Renderer/Converter → PDF/DOCX/PPTX/XLSX
                      ↓
                   VSCode → IntelliSense/Diagnostics
```

**Packages**:
- omniscript-parser (18 KB): Lexer, parser, AST
- omniscript-cli (21 KB): Commands, themes
- omniscript-converters (29 KB): PDF/DOCX/PPTX/XLSX
- omniscript-vscode (2,500 lines): IDE support
- omniscript-examples (2,700 lines): 25+ examples
- omniscript-site: Next.js docs & playground

**Block Types**:
@meta (metadata), @doc (markdown), @slide (presentations), @sheet (spreadsheets), @chart/@diagram/@code (v1.0).

**Integration Points**:
VSCode ↔ Parser (real-time), CLI ↔ Parser ↔ Converters, Website ↔ Parser (playground), Examples ↔ Validator ↔ Parser (CI).

## DEPLOYED ENVIRONMENTS
Production: https://omniscriptosf.github.io. NPM: omniscript-parser, omniscript-converters, omniscript-cli. VSCode: marketplace.visualstudio.com.

## VERSION CONTROL
Git. Follow .cursor/rules/universal.mdc. Small commits with conventional commits: feat:, fix:, docs:, chore:.

## ESSENTIAL COMMANDS
Parser: cd omniscript-core/parser && pnpm test. Converters: cd omniscript-converters && pnpm test. CLI: osf render file.osf. VSCode: cd omniscript-vscode && npm run compile. Website: cd omniscript-site && npm run dev. Examples: cd omniscript-examples && npm run validate.

## HEADER COMMENTS
Every file starts with four lines: 1) exact file location, 2) what this file does, 3) why this file exists, 4) RELEVANT FILES: comma-separated list. Never remove.

## WRITING STYLE
Short sentences. Plain English. Split long sentences, add blank line. Comment non-obvious code.

## SIMPLICITY
Simple is good. Complex is bad. Fewer lines better if clarity stays high. Prefer standard library.

## ROLES AND PERMISSIONS
Product Owner: sets scope (Alphin Tom). Developers: implement, review, release. Editor AI: edits files, no npm publish without approval. Autonomous Agents: refactors/tests only when requested, no production changes.

## UI PRINCIPLES
NOIR design system: black backgrounds, white text, accent colors. Simple, minimal, clear hierarchy. Accessibility before decoration.

## API CONVENTIONS
Library exports. Parser: parse(text: string) → Document. Converters: convert(document, options) → Buffer. CLI: osf render <file> --format <type>. Errors: throw Error with context.

## DATABASE PRINCIPLES
No database. File-based .osf format. Version control via git.

## SECURITY BASELINE
No secrets in code. Validate OSF syntax at parse. Pin dependencies, run npm audit. VSCode: sandbox user code.

## PRIVACY AND DATA RETENTION
Collect no user data. Examples use synthetic data. Log no sensitive info. No telemetry in core packages.

## LOGGING AND TELEMETRY
Levels: debug, info, warn, error. Parser errors include line/column. CLI: stderr for logs, stdout for output. VSCode: output channel, not console.

## ERROR HANDLING
Parse: fail fast with line number. Converters: validate before conversion. CLI: exit 1 on error, 0 on success. Helpful messages, no internal stacks in production.

## PERFORMANCE BUDGETS
Parser: 1000-line doc in <100ms. Converters: PDF in <2s. Website: LCP <2s on 4G. CLI: total <5s for typical doc.

## ACCESSIBILITY AND I18N
WCAG 2.1 AA for website. Keyboard navigation. Screen reader support. CLI/parser language-agnostic. Error messages in English only.

## FEATURE FLAGS AND ROLLOUT
New @blocks start experimental. Document in spec with version tag. Parser accepts but warns. Promote to stable after 2 releases.

## DEPENDENCIES
Prefer standard library. Parser: zero dependencies. Converters: pdfkit, docx, pptxgenjs, exceljs. VSCode: @types and vscode SDK only. Website: Next.js, React, Tailwind. Review licenses.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OmniScriptOSF/omniscriptosf.github.io](https://github.com/OmniScriptOSF/omniscriptosf.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
