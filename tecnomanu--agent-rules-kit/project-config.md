---
trigger: always_on
description: This repository contains **agent‑rules‑kit**, a CLI tool and template set that bootstraps Cursor rules for multiple stacks.
---

# Project Overview & Guidelines

This repository contains **agent‑rules‑kit**, a CLI tool and template set that bootstraps Cursor rules for multiple stacks.
The rules in this folder _govern the maintenance of this very project_.

| Topic       | Quick rule                                                                                                                                              |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CLI changes | Update code in `cli/services/` modules and `cli/index.js`; add unit and integration tests.                                                              |
| Templates   | Follow folder layout: `templates/global/`, `templates/stacks/<stack>/base/`, `templates/stacks/<stack>/architectures/`, `templates/stacks/<stack>/v*/`. |
| Config      | Keep `templates/kit-config.json` as single source of truth for globs, version ranges & default architectures.                                           |
| Releases    | Bump `package.json` + CHANGELOG. Tag using `vx.y.z`.                                                                                                    |
| Language    | **All code, documentation, and files must be written in English**, even if communicating with the agent in another language.                            |

## Language Policy

While you may interact with the agent in any language, all project files, code, comments, documentation, and rules must be written in English. This ensures consistency and maintainability across the project. When working with the Agent Rules Kit:

-   Write all code, comments, and docstrings in English
-   Create documentation files in English
-   Use English for variable names, function names, and other identifiers
-   Use English for commit messages and pull requests
-   Keep configuration files in English

For example, if you're communicating with the agent in Spanish, the agent should respond in Spanish, but any generated or modified code must still be in English.

# Global Coding Standards

These apply to **all code** in this repository (TypeScript, PHP, Markdown, JSON):

-   Use **ESLint + Prettier** for all JavaScript/TypeScript files; run `npm run lint` before committing.
-   PHP code must pass **PHP‑Stan level 8** and **Pint** auto‑format (run `composer pint`).
-   Markdown follows the GitHub Markdown style guide; wrap lines at 120 chars.
-   JSON files must be pretty‑printed with 2‑space indent.

> Any new template file must follow the same style conventions.

# Testing & CI

-   All CLI logic is covered by **Vitest** unit tests:
    -   `tests/cli/file-helpers.test.js` - Tests for file operations
    -   `tests/cli/config.test.js` - Tests for configuration handling
    -   `tests/cli/stack-helpers.test.js` - Tests for common stack functionality
    -   `tests/cli/nextjs-helpers.test.js` - Tests for Next.js specific functionality
-   Template‑generation is validated by snapshot tests (`tests/templates/`).
-   CI (GitHub Actions) runs `npm run test` and checks formatting.
-   Before publishing a new version, run `npm run test -- --update` to refresh snapshots if necessary.
-   Pre-commit and pre-push hooks run tests automatically using Husky.

# File Operations Guidelines

These guidelines apply to all file operations in the Agent Rules Kit project.
!Important: Al the system and files most be in english

 <!-- In the future: or using the translations system -->

## File Creation and Modification

-   Before creating a file, check if it already exists to avoid duplication
-   When modifying existing files, preserve the original formatting and style
-   If a file exists and there is no explicit instruction to overwrite, merge content instead of replacing
-   Ensure all new files have appropriate permissions
-   Add file headers with description and copyright information where applicable

## Template Files

-   Template files in the `templates/` directory should:
    -   Use clear, descriptive names
    -   Include meaningful comments
    -   Provide examples for customization
    -   Follow the established naming patterns
    -   Use proper file extensions

## Rule Files Structure

-   All rule files should follow a consistent structure:
    -   Title at the top
    -   Brief description of purpose
    -   Sections with clear headings
    -   Code examples where appropriate
    -   Version compatibility information if relevant

## Path Handling

-   Use path manipulation utilities (like Node.js `path` module) instead of string concatenation
-   Handle both relative and absolute paths correctly
-   Use platform-appropriate path separators
-   Normalize paths when comparing or storing them
-   Validate paths before file operations

## File I/O

-   Use asynchronous file operations when possible
-   Properly handle file operation errors
-   Close file handles after use
-   Use appropriate encoding for text files (UTF-8 preferred)
-   Implement proper error recovery for file operations

## Configuration Files

-   Keep configuration files in standard formats (JSON, YAML, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tecnomanu/agent-rules-kit](https://github.com/tecnomanu/agent-rules-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
