---
trigger: always_on
description: > This document is intended for developers taking over or contributing to the project.
---

# AGENTS.md — line-justice.nvim Handover Document

> This document is intended for developers taking over or contributing to the project.
> It captures architecture, design decisions, conventions, and practical guidance needed
> to get up to speed quickly.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Repository Layout](#2-repository-layout)
3. [Architecture & Data Flow](#3-architecture--data-flow)
4. [Configuration System](#4-configuration-system)
5. [Colour / Highlight System](#5-colour--highlight-system)
6. [Wrapped-Line Indicators](#6-wrapped-line-indicators)
7. [Public API](#7-public-api)
8. [Internal Helpers Reference](#8-internal-helpers-reference)
9. [statuscol.nvim Integration](#9-statuscolnvim-integration)
10. [Development Guidelines](#10-development-guidelines)
11. [Adding a New Feature — Step-by-step](#11-adding-a-new-feature--step-by-step)
12. [Theme Config Management](#12-theme-config-management)
13. [Adding a New Wrapped-Line Indicator Preset](#13-adding-a-new-wrapped-line-indicator-preset)
14. [Testing Checklist](#14-testing-checklist)
15. [Common Issues & Fixes](#15-common-issues--fixes)
16. [Git Conventions](#16-git-conventions)
17. [Dependencies](#17-dependencies)
18. [License](#18-license)

---

## 1. Project Overview

**line-justice.nvim** is a Neovim plugin that renders **both absolute and relative line numbers simultaneously** in the statuscolumn, solving a real-world friction point for pair programming, code reviews, and remote collaboration.

- The **left column** always shows the true (absolute) line number in the file.
- The **right column** shows the cursor-relative distance on every non-cursor line.
- The **cursor line** itself is highlighted distinctly (no relative number shown — it would always be `0`).
- Soft-wrapped continuation lines show a configurable indicator character instead of any numbers.

line-justice is a **statuscol.nvim segment provider**. It owns colour themes, highlight groups, number formatting, and wrapped-line rendering. It does **not** call `statuscol.setup()` — the user wires `require("line-justice").segment` into their own statuscol config. This eliminates any conflict with existing statuscol configurations.

**Minimum NeoVim version:** 0.10
**Language:** Lua
**License:** Apache 2.0
**Remote:** `https://github.com/zaakiy/line-justice.nvim`

---

## 2. Repository Layout

```
line-justice.nvim/
├── lua/
│   └── line-justice/
│       ├── init.lua                ← Plugin core: types, defaults, state, segment, highlights, public API
│       └── themes/
│           ├── init.lua            ← Theme registry (register, get, list, exists)
│           ├── horizon.lua         ← Built-in Horizon theme spec
│           ├── dawn.lua            ← Built-in Dawn theme spec
│           └── midnight.lua        ← Built-in Midnight theme spec
├── plugin/
│   └── line-justice.lua      ← Auto-sourced entry point; double-load guard + version check
├── examples/
│   ├── lazy-spec.lua         ← Full lazy.nvim plugin spec with annotated options
│   └── custom-theme.lua      ← Annotated example showing how to author and register a custom theme
├── README.md                 ← End-user documentation
├── SYSTEM_PROMPT.md          ← AI-assistant development context (gitignored)
├── LICENSE                   ← Apache 2.0
└── .gitignore
```

### Key facts

| File | Role |
|---|---|
| `lua/line-justice/init.lua` | Plugin core. Contains types, defaults, mutable render state, segment function, highlight resolution, and public API. |
| `lua/line-justice/themes/init.lua` | Theme registry. Exposes `register()`, `get()`, `list()`, `exists()`. Loads built-in specs lazily. |
| `lua/line-justice/themes/horizon.lua` | Horizon theme spec (`LineJusticeThemeSpec`). |
| `lua/line-justice/themes/dawn.lua` | Dawn theme spec — warm amber and rose tones. |
| `lua/line-justice/themes/midnight.lua` | Midnight theme spec — cool monochrome blue-greys. |
| `examples/custom-theme.lua` | Annotated example showing how to author and register a custom theme. |
| `plugin/line-justice.lua` | NeoVim auto-sources everything under `plugin/`. Sets `vim.g.loaded_line_justice` to prevent double-loading and checks `nvim-0.10`. |
| `examples/lazy-spec.lua` | Not loaded by NeoVim. Purely illustrative for users installing via lazy.nvim. |
| `SYSTEM_PROMPT.md` | Listed in `.gitignore`. Not shipped. Used for AI-assisted development sessions. |

---

## 3. Architecture & Data Flow

```
Module load  (require("line-justice"))
  └─ _state  initialised: { indicator_char = "", ready = false }
  └─ M.segment  assigned to _segment function (never nil from this point)

User calls lj.setup(opts)
  └─ vim.tbl_deep_extend("force", defaults, opts)  → config
  └─ M.themes.get(config.line_numbers.theme)        → theme_tbl
  └─ resolve_indicator(config.wrapped_lines)         → indicator_char
  └─ vim.o.number = true, vim.o.relativenumber = true
  └─ resolve_highlights(overrides, theme_tbl)
  │    → registers LineJustice{CursorLine,AbsoluteAbove,AbsoluteBelow,
  │      RelativeAbove,RelativeBelow,WrappedLine} via nvim_set_hl
  └─ nvim_create_autocmd("ColorScheme")
  │    → re-runs resolve_highlights on every :colorscheme change
  └─ _state.indicator_char = indicator_char

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zaakiy/line-justice.nvim](https://github.com/zaakiy/line-justice.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
