---
trigger: always_on
description: This document defines how autonomous workflow agents must work on the text0wnz repository. You should assume you already have a working environment with:
---

# Autonomous Agent Instructions for xero/text0wnz

This document defines how autonomous workflow agents must work on the text0wnz repository. You should assume you already have a working environment with:

- the repository checked out,
- Bun installed,
- project dependencies installed,
- dev tools available (eslint, prettier, playwright),

## Primary rule

- Always consult the docs in the repository's docs/ folder before making decisions. The docs are the authoritative source of project structure, patterns, build and test commands, and examples. Do not proceed with code changes unless you have read the relevant documentation files for the task.

## Required docs (always check these first)

- docs/architecture.md — system architecture, module responsibilities, event patterns, network protocol, build outputs.
- docs/building-and-developing.md — build system, Bun/Vite commands, environment variables, scripts, build output layout, and troubleshooting.
- docs/testing.md — Vitest/Playwright configs, test organization, mocking rules, and E2E guidance.
- docs/editor-client.md — client-side module responsibilities and API surface (Key Modules).
- docs/examples/ — sample artwork and example files to use as fixtures for tests.

## What to do before any work

1. Search the docs/ folder for relevant sections that apply to the task (module responsibilities, build/test commands, event/API patterns).
2. Load example files from docs/examples/ to use as test inputs for file-format work or UI rendering checks.
3. Ensure the environment is ready (bun install already run). If running Playwright E2E tests for the first time, run `bun test:install` to install browsers.

## Quick start (essential commands)

- bun bake
  - Builds the production application (generates dist/). Use this when producing a build artifact or validating the production bundle.
- bun www
  - Serves the built application from dist/ on localhost (default port 8060). Use this to validate production build behavior in a browser or to allow Playwright tests to run against a static server.
- bun server
  - Starts the backend collaboration server. Use this when changes affect server-side behavior or collaborative features.
- bun fix
  - Runs Prettier and then ESLint auto-fix. MUST be run before building or committing. Resolve any ESLint warnings/errors that remain after running it (do not ignore lint failures unless you explicitly explain why and update project config via PR).

## Behavioral patterns and code conventions (must follow)

- **Always use camelCase**. see docs /project-structure.md > Naming Conventions)
- Use Bun tooling and scripts defined in docs/building-and-developing.md for builds, runs, and scripts.
- Follow the project's established design patterns:
  - Revealing module pattern for modules.
  - Observer/event-driven architecture for canvas interactions (custom events such as onTextCanvasDown/onTextCanvasDrag/onTextCanvasUp).
  - Tool controller pattern with enable/disable methods and event listener registration/unregistration for each tool.
  - Command pattern for undo/redo operations (startUndo/endUndo).
- Web Worker and network security:
  - WebSocket work is performed in a Worker and requires a mandatory initialization sequence. Follow the worker init and trusted-URL construction patterns in docs.
- Font/glyph system:
  - Font loading is lazy and cached. Use the fontCache / lazyFont approach in the docs.
- Indentation/format:
  - Follow existing project style (tabs where present in codebase). Use `bun fix` to enforce formatting and linting automatically.

## Task categories

1) Features and enhancements
- Read architecture.md + editor-client.md to identify module boundaries and API surface.
- Implement code in the module that owns the functionality (follow module responsibilities from docs).
- Add unit tests that cover new behavior (see testing.md test structure). Use docs/examples/ as fixtures where applicable.
- Run `bun fix`, then `bun test:unit`. Ensure tests pass locally.
- Build the production bundle (`bun bake`) and validate with `bun www` where the change affects rendering or build outputs.
- Open a PR: create a feature branch, include a clear description, list the docs consulted, list test changes and CI considerations. Ensure CI passes.

2) Bug fixes
- Reproduce the bug using docs/examples/ if applicable; write a minimal test reproducing the bug (unit or e2e).
- Consult architecture.md or editor-client.md to understand intended behavior before changing code.
- Implement fix, add tests, run `bun fix`, run unit tests, run `bun bake` and validate if necessary.
- If the bug affects collaboration or server behavior, run the backend (`bun server`) and test multi-client behavior.
- Create a PR with regression tests included.

3) Unit tests (Vitest)
- Follow tests/ directory organization described in docs/testing.md.
- Use jsdom environment and existing setup files (tests/canvasShim.js, tests/setupTests.js).
- When mocking constructors, use `function` or `class` syntax (Vitest v4 requirement).
- Keep unit tests focused and small. Add fixtures from docs/examples/ where relevant.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xero/text0wnz](https://github.com/xero/text0wnz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
