---
trigger: always_on
description: Heximal is an HTML-first declarative reactive framework based on Lit and Web
---

# Heximal Agent Instructions

Heximal is an HTML-first declarative reactive framework based on Lit and Web
Components. It extends HTML with reactive templates, declarative custom
elements, lexically-scoped variables backed by signals, an expression language,
and elements to author dynamic documents in markup.

This document serves as the guide for AI agents working in this repository.

---

## Key Workflows & Tooling

### Wireit Orchestration (CRITICAL)

The project uses **[Wireit](https://github.com/google/wireit)** to manage build,
test, and dev server script execution, dependency graphs, and caching.

- **Never run manual builds before running tests**: Wireit automatically
  establishes dependency chains. For instance, package-level and root `test`
  tasks depend on `build`. Running `npm test` will automatically compile any
  modified TypeScript files or stale dependencies before executing tests.
- **Automatic Cache Reuse**: Wireit computes fingerprint hashes of inputs
  (source files, configs, dependency outputs). When inputs have not changed,
  Wireit immediately reuses cached outputs and skips the command. If `npm test`
  reports `Ran 0 scripts and skipped N in 0.1s`, tests are passing and
  up-to-date.
- **Running Tests**:
  - **Full test suite**: `npm test`
  - **Single package test suite**: `npm test -w @heximal/<package-name>` (e.g.
    `npm test -w @heximal/expressions` or `npm test -w @heximal/components`)
- **Building**:
  - **All packages**: `npm run build`
  - **Single package**: `npm run build -w <package-name>`
- **Formatting**:
  - Run Prettier: `npm run format`
  - Sync ignore files: `npm run ignore-sync` (uses `ignore-sync` to keep
    `.prettierignore` in sync with `.gitignore` and `.prettierignore-sync`).
- **Browser Test Runner Environment Note**:
  - Packages testing in the browser (`components`, `element`, `templates`) use
    `@web/test-runner` with Playwright Chromium.
  - In sandboxed CLI environments (such as sandboxed agent execution on macOS),
    Chromium may fail with Mach port permissions (`KERN_SUCCESS
    bootstrap_check_in`). Run tests unsandboxed (with permission / bypass
    sandbox) if browser tests fail to launch Chromium.
  - `@heximal/expressions` runs in Node (`node:test`) and does not require a
    browser.

---

## Monorepo Architecture & Packages

The repository is organized under npm workspaces (`packages/*`):

### 1. `packages/heximal` (`heximal`)

- Top-level convenience/umbrella package.
- Re-exports everything from `@heximal/components`, `@heximal/element`,
  `@heximal/expressions`, and `@heximal/templates`.
- Entrypoint: `index.js` (compiled from `src/index.ts`).

### 2. `packages/expressions` (`@heximal/expressions`)

- Expression parser and evaluator for JavaScript-like expressions.
- Hand-written, recursive-descent, precedence-climbing parser
  (`src/lib/parser.ts`, `tokenizer.ts`).
- Pluggable AST factories (`DefaultAstFactory` in `ast_factory.ts`,
  `EvalAstFactory` in `eval.ts`).
- Supports property access, identifiers, function/method calls, arrow functions,
  binary/unary operators, nullish coalescing (`??`), assignment (`=`), and
  pipeline operators (`|` and `|>`).
- Built-in null-safety (null/undefined subexpressions do not throw on property
  access).
- Tested with Node's native test runner (`node:test`, `node:assert`).

### 3. `packages/templates` (`@heximal/templates`)

- Declarative HTML `<template>` engine built on top of `lit-html` and
  `@heximal/expressions`.
- Core function: `prepareTemplate(templateElement, handlers, renderers,
  superTemplate)` transforms `<template>` elements into `lit-html` render
  functions.
- Expressions are delimited by `{{ expression }}`.
- Supports standard `lit-html` bindings:
  - Attributes: `attr={{ val }}`
  - Properties: `.prop={{ val }}`
  - Events: `@event={{ handler }}`
  - Boolean attributes: `?boolean={{ condition }}`
- Control flow via `<template type="...">` handlers:
  - `<template type="if" if="{{ condition }}">`
  - `<template type="repeat" repeat="{{ items }}">` (exposes `item` in scope)
- Template composition & inheritance:
  - Named blocks: `<template name="block-name">`
  - Super calls: `<template name="super">`
  - Sub-template calls: `<template call="..." data="...">` (by name or by
    reference).
- Tested with `@web/test-runner` and Playwright Chromium.

### 4. `packages/element` (`@heximal/element`)

- Declarative custom elements defined directly in HTML with
  `<h-define-element>`.
- Custom element name declared with `name="..."` (must contain a hyphen).
- Properties defined via `properties="..."` attribute or child `<h-prop
  name="..." type="..." reflect attribute="...">` elements.
- Adopted styles via child `<style type="adopted-css">`.
- Template via child `<template>`.
- Inheritance via `extends="base-element-name"`, supporting block overrides and
  `<template name="super">`.
- Reactivity backed by signals (`signal-polyfill` / `@lit-labs/signals`).
- Tested with `@web/test-runner`.

### 5. `packages/components` (`@heximal/components`)

- Core built-in elements and document-level scoping mechanism.
- Elements:
  - State & Scope:
    - `<h-var name="..." value="...">`: Signal-backed reactive variable.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elematic/heximal](https://github.com/elematic/heximal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
