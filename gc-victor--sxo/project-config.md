---
trigger: always_on
description: **Last Updated:** 2025-10-27
---

# AGENTS.md

**Last Updated:** 2025-10-27

## Purpose

This file is the onboarding manual for every AI assistant (Claude, Cursor, GPT, etc.) and every human who edits this repository. It encodes our coding standards, guard-rails, and workflow tricks so the _human 30 %_ (architecture, tests, domain judgment) stays in human hands.[^1]

[^1]: This principle emphasizes human oversight for critical aspects like architecture, testing, and domain-specific decisions, ensuring AI assists rather than fully dictates development.

---

## 0. Project Overview

This is a showcase project demonstrating the integration of SXO (a server-side JSX framework), reactive-component (for client-side islands), and basecoat-css (for component styling). It serves as a reference implementation and documentation for building applications with this modern stack.

### Components

| Component                                                                                               | Description                                                                   |
| ------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| **[SXO Framework](https://github.com/gc-victor/sxo/raw/refs/heads/main/README.md)**                     | Server-side JSX transformer and dev/prod servers with hot-reload              |
| **[reactive-component](https://github.com/gc-victor/reactive-component/raw/refs/heads/main/README.md)** | Lightweight reactive islands using `define()` API with signals                |
| **[basecoat-css](https://basecoatui.com/kitchen-sink/)**                                                | Pre-built component styles and design tokens                                  |
| **Tailwind CSS**                                                                                        | Utility-first CSS framework for custom styling                                |
| **Component Libraries**                                                                                 | `src/components/` (JSX components) and `src/components/html/` (HTML examples) |

### Golden Rule

**When unsure about implementation details or requirements, ALWAYS consult the developer rather than making assumptions.**

---

## 1. Non-Negotiable Golden Rules

| #       | AI _may_ do                                                                                                                                            | AI _must NOT_ do                                                                                                                                           |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **G-0** | Whenever unsure about something related to the project, ask the developer for clarification before making changes.                                     | ❌ Write changes or use tools when you are not sure about something project-specific, or if you don't have context for a particular feature/decision.      |
| **G-1** | Generate code **only inside** `src/` directory (specifically `src/components/`, `src/pages/`, `src/utils/`, `src/types/`) or explicitly pointed files. | ❌ Touch root-level config files ([biome.json](biome.json)`, [jsconfig.json](jsconfig.json), `[sxo.config.js](sxo.config.js)) without explicit permission. |
| **G-2** | Add/update **`AIDEV-NOTE:` anchor comments** near non-trivial edited code.                                                                             | ❌ Delete or mangle existing `AIDEV-` comments.                                                                                                            |
| **G-3** | Follow Biome lint/style configs ([biome.json](biome.json)). Use `npm run check` to validate.                                                           | ❌ Re-format code to any other style. Manual formatting not allowed.                                                                                       |
| **G-4** | For changes >300 LOC or >3 files, **ask for confirmation**.                                                                                            | ❌ Refactor large modules without human guidance.                                                                                                          |
| **G-5** | Stay within the current task context. Inform the dev if it'd be better to start afresh.                                                                | ❌ Continue work from a prior prompt after "new task" – start a fresh session.                                                                             |
| **G-6** | This is vanilla JSX (NOT React). Use native HTML semantics and standard DOM APIs.                                                                      | ❌ Use React patterns (`useState`, `useEffect`, etc.), React-specific props, or JSX runtime imports.                                                       |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gc-victor/sxo](https://github.com/gc-victor/sxo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
