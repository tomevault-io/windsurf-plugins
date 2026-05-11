---
trigger: always_on
description: A **1:1 port of TailwindCSS 4.x to PHP**. See [README.md](README.md) for scope and features.
---

# TailwindPHP - Development Guide

A **1:1 port of TailwindCSS 4.x to PHP**. See [README.md](README.md) for scope and features.

## Table of Contents

- [Critical Rules for LLMs](#critical-rules-for-llms)
  - [1. Never Modify Test Files](#1-never-modify-test-files-to-make-tests-pass)
  - [2. Always Fix at the Source](#2-always-fix-at-the-source)
  - [3. LightningCSS Functionality](#3-lightningcss-functionality)
  - [4. The _tailwindphp Directory](#4-the-_tailwindphp-directory)
  - [5. Companion Libraries](#5-companion-libraries-clsx-tailwind-merge)
  - [6. Plugin System](#6-plugin-system)
  - [7. Performance Optimizations](#7-performance-optimizations)
  - [8. Code Quality Tools](#8-code-quality-tools)
- [Test System](#test-system)
  - [Test Types](#test-types)
  - [Test Coverage Structure](#test-coverage-structure)
  - [Extraction Scripts](#extraction-scripts)
  - [Commands](#commands)
- [Development Workflow](#development-workflow)
  - [Standard Workflow](#standard-workflow)
  - [When Tests Fail](#when-tests-fail)
- [Project Structure](#project-structure)
- [Key Files](#key-files)
- [Public API](#public-api)
- [Common Patterns](#common-patterns)
- [Debugging Tips](#debugging-tips)
- [Current Status](#current-status)
  - [Core Tests](#core-tests-extracted-from-typescript-test-suites)
  - [Plugin Tests](#plugin-tests-srcplugin)
  - [Library Tests](#library-tests-src_tailwindphplib)
  - [API Coverage Tests](#api-coverage-tests-teststailwindphp)
  - [Unit Tests](#unit-tests-ported-from-typescript)
  - [Outside Scope](#outside-scope-0-tests---intentionally-empty)
  - [Implemented Features](#implemented-features)
  - [Port Deviation Markers](#port-deviation-markers)

---

## Critical Rules for LLMs

### 1. Never Modify Test Files to Make Tests Pass

**FORBIDDEN:** Changing assertions, expected values, or normalization logic in `*.test.php` files to make failing tests pass.

**ALLOWED:** Only fix test infrastructure bugs (parsing issues, file loading) - never change expected outputs.

### 2. Always Fix at the Source

When a test fails, the fix belongs in the **source code** (`src/*.php`), not in the test file. The tests define the contract - we must match TailwindCSS output exactly.

### 3. LightningCSS Functionality

TailwindCSS uses [lightningcss](https://lightningcss.dev/) (Rust) for CSS transformations. Since we can't use Rust in PHP, equivalent functionality goes in:

```
src/_tailwindphp/LightningCss.php
```

This includes:
- CSS nesting transformation (flattening `&` selectors)
- `@media` query hoisting
- `calc()` simplification
- Leading zero removal (`0.5` → `.5`)
- Transform function spacing
- Grid value normalization

### 4. The `_tailwindphp` Directory

`src/_tailwindphp/` contains PHP-specific helpers that are **NOT** part of the TailwindCSS port:
- `LightningCss.php` - CSS optimizations (lightningcss equivalent)
- `CssMinifier.php` - CSS minification
- `lib/` - Companion library ports (clsx, tailwind-merge, cva)

Everything else in `src/` should mirror TailwindCSS structure.

### 5. Companion Libraries (clsx, tailwind-merge)

We include PHP ports of [clsx](https://github.com/lukeed/clsx) and [tailwind-merge](https://github.com/dcastil/tailwind-merge) because they are essential companion libraries in the Tailwind ecosystem:

- **clsx** - Conditional class name construction (used in virtually every Tailwind project)
- **tailwind-merge** - Intelligent class conflict resolution (`p-2 p-4` → `p-4`)
- **cn()** - Combines both (the pattern popularized by [shadcn/ui](https://ui.shadcn.com/))

By including these, TailwindPHP provides a complete Tailwind development experience without requiring Node.js for anything.

**Important for LLMs**: These libraries live in `src/_tailwindphp/lib/` with their own namespace (`TailwindPHP\Lib\*`) to clearly separate them from the core TailwindCSS port. The public API functions (`cn`, `clsx`, `twMerge`, `twJoin`) are exposed in `src/index.php` under the main `TailwindPHP` namespace.

### 6. Plugin System

TailwindPHP includes PHP ports of official TailwindCSS plugins:

- **@tailwindcss/typography** - The prose class for beautiful typographic defaults
- **@tailwindcss/forms** - Form element reset and styling utilities

These are 1:1 ports following the exact same logic as the JavaScript originals. The plugin system architecture:

```
src/plugin.php                    # Core plugin system
├── PluginInterface               # Contract for plugins (getName, __invoke, getThemeExtensions)
├── PluginAPI                     # API passed to plugins (addBase, addUtilities, addComponents, etc.)
└── PluginManager                 # Registry and execution of plugins

src/plugin/plugins/
├── typography-plugin.php         # @tailwindcss/typography port
└── forms-plugin.php              # @tailwindcss/forms port

test-coverage/plugins/
├── typography/                   # Extracted typography tests
│   ├── summary.json
│   └── tests/*.json
└── extract-typography-tests.php  # Extraction script
```

**PluginAPI Methods** (same as TailwindCSS):
- `addBase(array $css)` - Add base styles
- `addUtilities(array $utilities)` - Add static utilities
- `matchUtilities(array $utilities, array $options)` - Add functional utilities with values

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dnnsjsk/tailwindphp](https://github.com/dnnsjsk/tailwindphp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
