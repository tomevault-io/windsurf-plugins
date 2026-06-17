---
trigger: always_on
description: > This guide covers the whole monorepo. Sections marked **apps/scream-code** are app-specific; the rest apply to all workspace packages.
---

# scream-code Development Guide

> This guide covers the whole monorepo. Sections marked **apps/scream-code** are app-specific; the rest apply to all workspace packages.

## Table of Contents

1. [Workspace Overview](#workspace-overview)
2. [Code Quality & Style](#code-quality--style)
3. [TUI Sanitization](#tui-sanitization)
4. [Testing Guidance](#testing-guidance)
5. [Commands & Workflow](#commands--workflow)
6. [TUI File Layout (apps/scream-code)](#tui-file-layout-apps-scream-code)
7. [Module Responsibilities (apps/scream-code)](#module-responsibilities-apps-scream-code)
8. [ScreamTUI Internal Sections (apps/scream-code)](#screamtui-internal-sections-apps-scream-code)
9. [Where New Features Go (apps/scream-code)](#where-new-features-go-apps-scream-code)
10. [TUI Coding Conventions (apps/scream-code)](#tui-coding-conventions-apps-scream-code)
11. [How to Set Themes (apps/scream-code)](#how-to-set-themes-apps-scream-code)
12. [MCP (apps/scream-code)](#mcp-apps-scream-code)
13. [Slash Commands (apps/scream-code)](#slash-commands-apps-scream-code)
14. [Agent-Core Mechanisms](#agent-core-mechanisms)
15. [General Coding Requirements](#general-coding-requirements)

---

## Workspace Overview

### Packages

| Package | Path | Responsibility |
| ----------------------- | ---------------------------------------------------- | ---------------------------------------------------------------------- |
| `agent-core` | `packages/agent-core/` | Agent runtime: turn loop, session, tools, MCP client, compaction, memory, goal/wolfpack |
| `ltod` | `packages/ltod/` | Multi-provider LLM client with streaming support |
| `jian` | `packages/jian/` | Execution environment abstractions (filesystem, process, sandbox) |
| `node-sdk` | `packages/node-sdk/` | Node.js SDK (`ScreamHarness`, `Session`) consumed by the app |
| `memory` | `packages/memory/` | Cross-session memory store and scoring |
| `telemetry` | `packages/telemetry/` | Telemetry, crash reporting, usage metrics |
| `config` | `packages/config/` | Platform configuration, identity, model aliases |
| `migration-legacy` | `packages/migration-legacy/` | Legacy data migration — **deprecated, do not expand** |
| `apps/scream-code` | `apps/scream-code/` | CLI and terminal UI application (`scream` command) |

### Terminology

- When the user says **"agent"** or **"session"**, they mean the `packages/agent-core` runtime (`Session`, `Agent`, turn loop), not the assistant.
- **"app"** / **"TUI"** / **"CLI"** refers to `apps/scream-code`.
- **"SDK"** refers to `@scream-cli/scream-code-sdk` exported from `packages/node-sdk`.
- **"LLM layer"** refers to `packages/ltod`.
- **"memory"** refers to `packages/memory` task-experience records.

### Cross-package Import Rules

- `apps/scream-code` must use core capabilities **only through `@scream-cli/scream-code-sdk`**. Never import `@scream-cli/agent-core` directly in app code.
- `packages/agent-core` must not depend on `apps/scream-code`.
- Prefer package-local imports. When crossing packages, import from the package's public `index.ts` or documented subpaths.
- For Node built-ins, prefer namespace imports: `import * as fs from 'node:fs/promises'`, `import * as path from 'node:path'`.

---

## Code Quality & Style

### TypeScript

- Avoid `any`. If unavoidable, add a short comment explaining why.
- Do **not** introduce new `ReturnType<>` usage for new code; prefer explicit type names. Existing uses (e.g., timer IDs) should migrate to named aliases when touched.
- Avoid inline type imports such as `import('pkg').Type` or `import('./module').Type`. Use top-level imports.
- Optional object properties: pass `undefined` directly — do not use conditional spread.
- Internal methods with only a single parameter should not be turned into options objects just for stylistic uniformity.
- Except for a package's own public `index.ts`, internal `index.ts` barrels should prefer `export * from './module'`.

### Classes

- The current codebase uses `private readonly` for internal class state. Keep this style within a file; do not mix `private readonly` and native `#private` fields in the same component.
- Constructor parameter properties are fine (e.g., `constructor(private readonly host: Host)`).
- Leave externally accessible members bare (no `public` keyword).

### Promises & Async

- New code should prefer `Promise.withResolvers()` when it simplifies control flow. Do not refactor existing `new Promise` code purely for style.
- In Bun contexts, prefer `await Bun.sleep(ms)` over `new Promise(r => setTimeout(r, ms))`.

### Prompts & Static Copy

- Tool descriptions and system prompts live in `.md` files next to the code that uses them.
- Import them through the project's raw-text loader, e.g.:
  ```ts
  import DESCRIPTION from './tool.md';
  ```
  Do not inline multi-line prompts as template literals.
- UI copy, option labels, help text, and dialog titles should stay next to the component or command that uses them. Do not centralize them into a global "copy constants" module.

### Logging

- **Never use `console.log` / `console.warn` / `console.error` in TUI components or render paths** — it corrupts terminal rendering.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LIUTod/scream-code](https://github.com/LIUTod/scream-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
