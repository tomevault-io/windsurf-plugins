---
trigger: always_on
description: Transforms `<For>` components into `forBlock()` runtime calls.
---


# Roqa Project Overview

Roqa is a **compile-time reactive UI framework** for building web components with JSX syntax. Unlike virtual DOM frameworks, Roqa compiles JSX into highly optimized vanilla JavaScript that directly manipulates the DOM.

## Core Concepts

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                          Build Time                             │
│   ┌──────────┐    ┌────────────────┐    ┌───────────────────┐   │
│   │  .jsx    │───▶│  Vite Plugin   │───▶│  Compiled Output  │   │
│   │  files   │    │  (roqa)     │    │  (vanilla JS)     │   │
│   └──────────┘    └────────────────┘    └───────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│                           Runtime                              │
│   ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐   │
│   │   template   │  │    cell      │  │  defineComponent   │   │
│   │  (DOM clone) │  │  (reactive)  │  │  (web component)   │   │
│   └──────────────┘  └──────────────┘  └────────────────────┘   │
└────────────────────────────────────────────────────────────────┘
```

### Key Primitives

| Primitive          | Purpose                      | Compiled Form                               |
| ------------------ | ---------------------------- | ------------------------------------------- |
| `cell(value)`      | Create reactive state        | `{ v: value, e: [] }`                       |
| `get(cell)`        | Read cell value              | `cell.v`                                    |
| `put(cell, value)` | Write without notifying      | `cell.v = value`                            |
| `set(cell, value)` | Write and notify subscribers | `{ cell.v = value; /* inlined updates */ }` |
| `bind(cell, fn)`   | Subscribe to changes         | Inlined at `set()` call sites               |
| `notify(cell)`     | Trigger all subscribers      | Loops over `cell.e` and calls each          |

---

## Project Structure

```
packages/
├── roqa/          # Core framework
│   └── src/
│       ├── compiler/ # JSX → JS compiler (build-time)
│       └── runtime/  # Minimal runtime helpers
└── vite-plugin/      # Vite integration
```

---

## Compiler Deep Dive

> ⚠️ **CRITICAL FOR AI MODELS**: The compiler has a **strict 4-phase pipeline**. Each phase has specific inputs and outputs. Do not conflate phases.

### Compiler Pipeline

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         COMPILATION PIPELINE                                │
│                                                                             │
│   Source (.jsx)                                                             │
│        │                                                                    │
│        ▼                                                                    │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  PHASE 1: PARSE                                                     │   │
│   │  ────────────────                                                   │   │
│   │  File: parser.js                                                    │   │
│   │  Input: JSX source code string                                      │   │
│   │  Output: Babel AST                                                  │   │
│   │  Uses: @babel/parser with jsx plugin                                │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│        │                                                                    │
│        ▼                                                                    │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  PHASE 2: VALIDATE                                                  │   │
│   │  ─────────────────                                                  │   │
│   │  File: transforms/validate.js                                       │   │
│   │  Input: Babel AST                                                   │   │
│   │  Output: AST (unchanged) or throws error                            │   │
│   │  Purpose: Reject unsupported PascalCase components                  │   │
│   │  Note: <For> and <Show> are the ONLY allowed PascalCase components  │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│        │                                                                    │
│        ▼                                                                    │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  PHASE 3: GENERATE OUTPUT                                           │   │
│   │  ────────────────────────                                           │   │
│   │  File: codegen.js (orchestrator)                                    │   │
│   │  Input: Original source + AST                                       │   │
│   │  Output: Transformed source code                                    │   │
│   │                                                                     │   │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roqajs/roqa](https://github.com/roqajs/roqa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-11 -->
