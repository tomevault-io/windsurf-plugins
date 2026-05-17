---
trigger: always_on
description: This document is intended for "AI coding agents/Agent tools" (such as the built-in Agent in this repository, various automated PR bots, and IDE-based intelligent coding assistants). The goal is to help agents quickly and correctly complete changes without disturbing human collaborators, while maintaining unified repository style, maintainability, and verifiability.
---

# Chaterm Project AGENTS.md

This document is intended for "AI coding agents/Agent tools" (such as the built-in Agent in this repository, various automated PR bots, and IDE-based intelligent coding assistants). The goal is to help agents quickly and correctly complete changes without disturbing human collaborators, while maintaining unified repository style, maintainability, and verifiability.

— If you are a human contributor, you can also refer to this document's workflow and acceptance checklist.

## Project Overview

- Tech Stack: Electron + Vite + Vue 3 + TypeScript
- Build & Run: electron-vite, electron-builder
- Code Structure:
  - `src/main`: Electron main process logic (windows, updates, SSH, storage, etc.)
  - `src/preload`: Preload scripts (secure bridge)
  - `src/renderer`: Frontend rendering layer (Vue, Pinia, Router, i18n)
  - `src/main/agent`: In-project AI Agent capabilities (LLM Provider, context, tools, etc.)
  - `src/main/storage/db`: Local DB (better-sqlite3), migrations and services

## Code Structure Quick Reference

- Main process entry: `src/main/index.ts`
- Renderer entry: `src/renderer/src/main.ts`
- Preload declarations: `src/preload/index.ts`, `src/preload/index.d.ts`
- Routing & State:
  - Router `src/renderer/src/router/index.ts`
  - Store `src/renderer/src/store/*.ts`
- Agent capabilities: `src/main/agent/*` (providers, core, integrations, utils, shared, etc.)
- DB services: `src/main/storage/db/*` (connections, tables, migrations, services)
- Build config: `electron.vite.config.ts` (aliases, proxy, sourcemap, component auto-import, etc.)

## Behavioral Rules

### Rule 1 — Think Before Coding
State assumptions explicitly. If uncertain, ask rather than guess. Push back when a simpler approach exists. Stop when confused — name what's unclear.

### Rule 2 — Simplicity First
Minimum code that solves the problem. No speculative features. No abstractions for single-use code. If a senior engineer would call it overcomplicated, simplify.

### Rule 3 — Surgical Changes
Touch only what you must. Don't "improve" adjacent code, comments, or formatting. Don't refactor what isn't broken. Match existing style.

### Rule 7 — Surface Conflicts, Don't Average Them
If two existing patterns contradict, pick one (more recent / more tested), explain why, and flag the other for cleanup. Don't blend conflicting patterns.

### Rule 8 — Read Before You Write
Before adding code in a file, read its exports, the immediate caller, and any obvious shared utilities. "Looks orthogonal to me" is dangerous — if unsure why code is structured a certain way, ask.

### Rule 12 — Fail Loud
"Completed" is wrong if anything was skipped silently. "Tests pass" is wrong if any were skipped. Default to surfacing uncertainty, not hiding it.

## Basic Principles (Required Reading for Agents)

- Small and Precise: Only modify files directly related to requirements, avoid "incidental" refactoring of unrelated code.
- Maintain Style: Follow ESLint + Prettier and existing naming, directory, and import alias conventions.
- Type First: Complete TypeScript types, use `any` cautiously; define types for new APIs/IPC.
- Include Tests: Core logic that is added or modified should have Vitest unit tests; end-to-end flows can add Playwright test cases.
- Don't Break Contracts: Respect existing IPC, Store, and service interfaces; new capabilities should use an "optional extension" approach.
- Sync Documentation: User-visible behavior changes or new capabilities should update `README.md/README_zh.md` or related comments.

## Change Scope Options (When Adding Features)

- Related-Only (Default/Mandatory):
  - Ignore existing issues unrelated to current functionality (code smells, historical naming, style inconsistencies, readability, etc.), do not modify them arbitrarily.
  - Only modify code and configuration directly related to "new features" or "issues explicitly to be fixed now".
  - If important issues are found (security/stability/build blockers), record them in the PR's "Known issues/Follow-ups" section, open an Issue if necessary; do not handle them in this change.
  - Lint/formatting auto-fixes are limited to files involved in this change, avoid cascading changes.

- Broadened-Refactor (Requires Maintainer Approval):
  - Only use after explicit maintainer approval; allows cascading refactoring or batch fixes of unrelated issues.
  - PR title must include prefix `[refactor-approved]`, description must list impact scope, migration/rollback plan, and verification method.

## Submission Process (Command Quick Reference)

- Install Dependencies:
  - `node scripts/patch-package-lock.js`
  - `npm install`
- Development & Debugging: `npm run dev`
- Code Checks:
  - Format `npm run format`
  - Lint fix `npm run lint`
  - Type check `npm run typecheck`
- Testing:
  - Unit tests `npm test`
  - E2E `npm run test:e2e` / `npm run test:e2e:headed`
- Build:
  - General `npm run build`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chaterm/Chaterm](https://github.com/chaterm/Chaterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
