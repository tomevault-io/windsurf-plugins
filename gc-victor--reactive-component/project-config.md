---
trigger: always_on
description: **Project**: `reactive-component` • **Last Updated**: 2026-04-08 (UTC)
---

# AGENTS.md

**Project**: `reactive-component` • **Last Updated**: 2026-04-08 (UTC)

## Purpose

This file is the onboarding manual for every AI assistant (Claude, Cursor, GPT, etc.) and every human who edits this repository. It encodes our coding standards, guard-rails, and workflow tricks so the **human 30%** (architecture, tests, domain judgment) stays in human hands.[^1]

[^1]: This principle emphasizes human oversight for critical aspects like architecture, testing, and domain-specific decisions, ensuring AI assists rather than fully dictates development.

---

## 0. Project Overview

### Introduction

`reactive-component` is a library and example workspace for building robust, accessible Web Components using the ReactiveComponent library. It focuses on component-driven development, predictable reactive state management, and clean separation between structure (HTML), behavior (TypeScript/JavaScript), and style (CSS).

### Core Components

| Component         | Description                                                                      |
| ----------------- | -------------------------------------------------------------------------------- |
| **reactive-core** | Base ReactiveComponent class and reactive state engine                           |
| **components**    | Reusable Web Components built with ReactiveComponent                             |
| **examples**      | Example usages demonstrating bindings, computed state, refs, and custom handlers |
| **docs**          | Documentation and best practices for the framework                               |
| **tooling**       | Linting, formatting, type-checking, and build tools                              |

### Golden Rule

> **When unsure about implementation details or requirements, ALWAYS consult the developer rather than making assumptions.**

---

## 1. Non-Negotiable Golden Rules

| #       | AI _may_ do                                                                                                                 | ❌ AI _must NOT_ do                                                                                          |
| ------- | --------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **G-0** | Whenever unsure about something that's related to the project, ask the developer for clarification before making changes.   | ❌ Write changes or modify configs when you are not sure about project-specific expectations or conventions. |
| **G-1** | Generate code **only inside** relevant source directories (`src/`, `components/`, `examples/`) or explicitly pointed files. | ❌ Touch `tests/` or specification files without explicit instruction (humans own tests & specs).            |
| **G-2** | Follow the project's configured formatter and linter (Biome via `biome.json`). Use configured scripts.                      | ❌ Re-format code to any other style or add competing formatters/linters.                                    |
| **G-3** | For changes **>300 LOC** or **>3 files**, ask for confirmation.                                                             | ❌ Refactor large modules without human guidance.                                                            |
| **G-4** | Stay within the current task context. Inform the dev if it'd be better to start afresh.                                     | ❌ Continue work from a prior prompt after "new task" – start a fresh session.                               |

---

## 2. Build, Test & Utility Commands

### Introduction

Use package manager scripts for consistency (they ensure correct environment variables and configuration). If a script is missing, request guidance instead of guessing.

### Commands

```bash
# Dependencies
pnpm ci                  # install dependencies (CI-friendly)
pnpm install             # local install

# Development
pnpm build               # build library/components
pnpm dev                 # local development (avoid long-running servers in non-interactive contexts)

# Quality Assurance
pnpm test                # run unit tests (Vitest)
pnpm lint                # lint (Biome)
pnpm format              # format (Biome)
pnpm typecheck           # TypeScript type checking
```

> **Note**: Ensure correct CWD (project root) before running commands. Avoid launching watchers/servers in contexts that cannot be terminated.

---

## 3. Coding Standards

### Standards Table

| Aspect                | Standard                                                                                                                                   |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **Language**          | TypeScript preferred; modern JavaScript acceptable for small, isolated examples                                                            |
| **Web Components**    | Custom Elements v1; use Shadow DOM as needed; follow platform standards                                                                    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gc-victor/reactive-component](https://github.com/gc-victor/reactive-component) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
