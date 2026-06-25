---
trigger: always_on
description: provides deep context for a specific area of the codebase. Activate the appropriate
---

<!--
MAINTENANCE: Update this file when:
- Adding/removing npm scripts in package.json or targets in Makefile
- Changing the runtime flow (shell, remote host, service worker, php worker)
- Modifying the Moodle bundle format, manifest schema, or storage model
- Changing deployment assumptions for GitHub Pages or other static hosting
- Adding new conventions for blueprints, extensions, or persistent state
- Updating upstream project references (WordPress Playground, Omeka S Playground)
- Adding or removing agent skills under .agents/skills/
-->

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

Moodle Playground runs a Moodle site entirely in the browser using WebAssembly.
It follows the same product shape as `omeka-s-playground`:

1. Shell UI: `index.html` and `src/shell/main.js`
2. Runtime host: `remote.html` and `src/remote/main.js`
3. Request routing: `sw.js` and `php-worker.js`
4. PHP/Moodle runtime: `src/runtime/*` + generated assets under `assets/moodle/`

The Moodle core is extracted from a prebuilt ZIP bundle into Emscripten MEMFS (in-memory)
at boot. All files — core and mutable state — live in writable MEMFS. The runtime is fully
ephemeral — all state is lost when the browser tab closes or the page is reloaded.

## Related Projects and Upstream References

This project builds on WordPress Playground (`@php-wasm/*`) for the PHP WASM runtime and
Omeka S Playground for the shell/remote/sw/worker architecture pattern. Before inventing a
solution, check if either upstream already solved the same problem.

For full details: @.agents/references/upstream-projects.md

## Specialist Agent Skills

This project includes domain-expert agent skills under `.agents/skills/`. Each skill
provides deep context for a specific area of the codebase. Activate the appropriate
skill when working in its domain — the skill file contains API references, checklists,
known pitfalls, and conventions that are not repeated elsewhere in this document.

| Skill | Directory | When to use |
|-------|-----------|-------------|
| **Moodle Internals** | `@.agents/skills/moodle-internals/SKILL.md` | Moodle APIs, plugin system, database schema, install/upgrade lifecycle, config settings, course structure, user management, enrollment, MUC caching, SQLite compatibility, patch layout, bootstrap fragile areas |
| **WP Playground & php-wasm** | `@.agents/skills/wp-playground-php-wasm/SKILL.md` | `@php-wasm/web` and `@php-wasm/universal` APIs, PHP instance lifecycle, `php.run()` execution model, filesystem operations, `setPhpIniEntries()`, request/response conversion, `php-compat.js` adapter, outbound PHP networking, php.ini configuration |
| **WASM & Browser Runtime** | `@.agents/skills/wasm-browser-runtime/SKILL.md` | WASM crashes and memory limits, Emscripten MEMFS, service worker routing and caching, Web Worker communication, crash recovery, GitHub Pages subpath deployment, browser storage constraints, Firefox SW bundling |
| **Blueprint Provisioning** | `@.agents/skills/blueprint-provisioning/SKILL.md` | Blueprint JSON format, step handlers, executor engine, resource resolution, PHP code generation, plugin/theme installation, constant substitution, adding new step types |
| **Unit Testing** | `@.agents/skills/unit-testing/SKILL.md` | Writing and reviewing unit tests with `node:test`, mocking `php.run()` and MEMFS, testing PHP code generators, service worker helpers, runtime utilities, test organization conventions |
| **E2E Testing (Playwright)** | `@.agents/skills/e2e-playwright/SKILL.md` | Browser-based end-to-end tests with Playwright, WASM boot waiting strategies, iframe navigation, blueprint execution verification, shell UI interaction, debugging flaky tests |

### Additional references

| Reference | Location | Content |
|-----------|----------|---------|
| **Testing & CI/CD** | `@.agents/references/testing-and-ci.md` | Test suite inventories, CI/CD pipeline, Biome linting, Firefox compatibility, manual validation |
| **Upstream Projects** | `@.agents/references/upstream-projects.md` | WordPress Playground and Omeka S Playground details, when to consult each |

### Skill activation guidelines

1. **Read the skill file** when entering its domain — it contains the authoritative
   reference for conventions and known issues in that area.
2. **Cross-reference skills** when a change spans domains. For example, adding a new
   blueprint step that installs a plugin touches both `blueprint-provisioning` and
   `moodle-internals` (plugin type system, upgrade lifecycle).
3. **Follow the checklists** at the end of each skill file before submitting changes.
4. **Do not duplicate** skill content in this file — AGENTS.md provides the architectural
   overview; skills provide the deep domain knowledge.

## Build System

This project uses npm, esbuild, and a small Makefile workflow.

### Requirements

- Node.js 18+
- npm
- Python 3
- Git
- PHP 8.3 with `pdo_sqlite` for `make up-local`
- Composer — required to build Moodle **5.1+** bundles (`MOODLE_501_STABLE`,
  `main`, …). Since 5.1 `vendor/` is no longer committed upstream, so
  `scripts/build-moodle-bundle.sh` runs `composer install` for those branches.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ateeducacion/moodle-playground](https://github.com/ateeducacion/moodle-playground) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
