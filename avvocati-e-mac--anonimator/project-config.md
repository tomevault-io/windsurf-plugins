---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Desktop Electron application for **local pseudonymization** of Italian legal documents (PDF, DOCX, ODT, TXT, images). Target users are lawyers with low technical skills. All processing happens **offline** - no network access during document processing to comply with GDPR and professional secrecy requirements.

**Tech Stack:** Electron + React 18 + TypeScript (strict mode)

## Session Memory

Before starting any work, read the latest file in `sessioni/` to understand what was done in previous sessions, which decisions were made, and what the current state of the project is.

After completing significant work, update or create a new session file in `sessioni/` documenting decisions, files changed, and next steps. Session files are named: `sessione_NNN_faseN.md` (e.g. `sessione_001_fase1.md`)

**Template sessione obbligatorio:**
```markdown
# Sessione NNN — [Titolo breve]
**Data:** YYYY-MM-DD
**Versione:** x.y.z

## Obiettivo
## Decisioni prese
## File modificati
## Problemi noti / TODO prossima sessione
```

## Session Startup Checklist

Prima di iniziare qualsiasi lavoro, eseguire questi controlli:

1. [ ] Letto il file più recente in `sessioni/`
2. [ ] Verificato che `npm run typecheck` passi (nessun errore preesistente)
3. [ ] Controllato `git status` (nessun file uncommitted non intenzionale)
4. [ ] Gemini CLI disponibile? `command -v gemini` (opzionale — skip se non presente)

Read-only operations (`cat`, `grep`, `git log`, `git diff`, `git status`, `npm run typecheck`) do **NOT** require user confirmation — execute immediately. Write/delete/commit operations require confirmation only if not part of an already-approved plan.

## AI Agent Roles

### Claude Code (primary)
- Writes/modifies all repo files, runs build/test, controlled refactoring, debugging
- Implements the roadmap phase by phase, stops at end of each phase for user confirmation
- Updates `sessioni/` files after each significant work session

### Gemini CLI (secondary — research + code drafting, does NOT commit files)

Use Gemini CLI for targeted research and isolated code drafting when available.

**Pre-flight check — verificare disponibilità prima di usarlo:**
```bash
if ! command -v gemini &> /dev/null; then
  echo "Gemini CLI non disponibile — skip, procedi senza."
fi
```
Se il comando non è disponibile, **non usare Gemini CLI** e procedere normalmente con Claude Code. Non interrompere il lavoro per installarlo.

**Quando usare Gemini CLI** (solo se disponibile):
- Researching a specific library API or finding the correct method signature
- Evaluating edge cases or alternative implementations
- Checking model availability on HuggingFace or verifying ONNX compatibility
- Drafting a well-scoped, isolated piece of code (single parser, utility function, regex pattern, standalone React component with no IPC dependencies)

**How to invoke Gemini CLI from Claude Code:**
```bash
gemini -p "Your research question or code drafting request here"
```

Example use cases:
- `gemini -p "What is the correct Transformers.js pipeline syntax for token-classification with Italian_NER_XXL_v2 ONNX model?"`
- `gemini -p "How does adm-zip handle UTF-8 XML content in DOCX files on Windows?"`
- `gemini -p "What are the OCR confidence thresholds in tesseract.js v5 and how to read them?"`
- `gemini -p "Write a TypeScript function that strips Markdown syntax and returns plain text, using only Node.js built-ins"`

**Rules for code drafting via Gemini:**
- Only use Gemini for **isolated, well-specified** tasks (single function, single parser, standalone utility)
- Claude Code must **always review, adapt to project conventions, and commit** the result — never paste Gemini output directly
- Gemini output must comply with TypeScript strict mode, project naming conventions, and IPC security rules
- Document Gemini's contribution in the relevant session file in `sessioni/`

## Critical Rules (Non-Negotiable)

Before making any changes, understand these absolute requirements have priority over any other best practices:

1. **ZERO network calls** during document processing. No external APIs, telemetry, or crash reporting during analysis/anonymization. Only exception: initial model download and optional update check (outside processing flow).
2. **Electron Security:**
   - Renderer: `nodeIntegration: false`, `contextIsolation: true`, `sandbox: true`
   - Use only `contextBridge` + `ipcRenderer.invoke/on` for communication
   - Validate ALL IPC inputs in Main process with Zod
3. **TypeScript strict mode everywhere.** No implicit `any` types.
   - **NO Type Cheating:** `// @ts-ignore`, `// @ts-expect-error`, and type assertions `as any` are **STRICTLY FORBIDDEN**. Fix the actual TypeScript interfaces — never silence the compiler.
4. **Incremental development:** One feature/fix at a time. STOP at end of each logical unit and wait for user confirmation before proceeding.
5. **Git commits** before any significant modifications to existing files. Use Conventional Commits:
   - `feat:` new feature visible to the user
   - `fix:` bug fix

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [avvocati-e-mac/anonimator](https://github.com/avvocati-e-mac/anonimator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
