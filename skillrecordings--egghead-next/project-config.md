---
trigger: always_on
description: **Your primary task: Meticulously update the `./cursor/rules/_user-<username>.mdc` file. Use ONLY information found within the current project's files and documentation.**
---

# Update User Project Context File: .cursor/rules/_user-<username>.mdc

**Your primary task: Meticulously update the `./cursor/rules/_user-<username>.mdc` file. Use ONLY information found within the current project's files and documentation.**
## Determine GitHub Username

Run the following to capture your authenticated GitHub username:
```bash
GH_USER=$( (gh api user --jq .login 2>/dev/null || git config user.username || git config user.name) \
  | tr '[:upper:]' '[:lower:]' \
  | xargs \
  | tr ' ' '-' \
  | tr -d '\n' )
```
Then replace `<username>` in the file path with `$GH_USER`.

## File Existence Check

Quickly list potential configuration files and docs:

```bash
ls . docs
```

Use the output to see which files are available to read.

## 1. Core Objective

Your goal is to populate the `./cursor/rules/_user-<username>.mdc` file. This file must accurately reflect the project's current configuration, technology stack, structure, standards, and tooling, serving as a central project reference.

## 2. Information Gathering (Strictly from Project Sources)

Follow these steps precisely to gather the necessary information:

### Step 1: Analyze Project Root & Configuration Files:
- List all files and directories in the project root.
- Identify and thoroughly read relevant configuration files. Examples include (but are not limited to):
    - Package management: `package.json`, `pnpm-lock.yaml`, `yarn.lock`, `pyproject.toml`, `poetry.lock`, `go.mod`, `pom.xml`, `build.gradle`, `Gemfile`, `composer.json`
    - Build tools & bundlers: `webpack.config.js`, `vite.config.js`, `rollup.config.js`, `tsconfig.json` (check for `compilerOptions`, `extends`, etc.)
    - Framework specific: `next.config.js`, `angular.json`, `vue.config.js`, `svelte.config.js`
    - Linters & formatters: `.eslintrc.js`, `.eslintignore`, `.prettierrc.js`, `.prettierignore`, `biome.json`, `ruff.toml`, `.editorconfig`
    - CI/CD: Files in `.github/workflows/`, `gitlab-ci.yml`, `Jenkinsfile`, `Dockerfile`
    - Testing: `jest.config.js`, `vitest.config.js`, `cypress.json`, `playwright.config.js`
- From these files, extract details about dependencies (and their versions), scripts (build, dev, test), language versions, tool configurations, and project settings.

### Step 2: Review Documentation and Key Code Directories:
- Thoroughly read all files located within the `docs/` directory (and its subdirectories). Prioritize `README.md`, `CONTRIBUTING.md`, architecture diagrams, technical specifications, or any documents describing the project's setup, conventions, and purpose.
- Examine the structure of main source directories (e.g., `src/`, `app/`, `lib/`, `packages/`) to understand module organization and identify key architectural patterns.

## 3. Updating `./cursor/rules/_user-<username>.mdc`

### Step 3: Populate the Template:
- Open the `./cursor/rules/_user-<username>.mdc` file for editing.
- Using the information gathered in Steps 1 and 2, meticulously fill in each section of the template provided below.
- **Crucial:** Adhere strictly to the template's structure and placeholders. Provide specific examples and details where requested (e.g., actual library names, version numbers, file paths).
- **Handling Missing Information:** If, after thorough investigation, specific information required by the template cannot be definitively found within the project's files or `docs/` directory, explicitly state `[Information not found in project context]` or `[N/A]` in the corresponding field. **ABSOLUTELY DO NOT invent, infer, or use external knowledge.** It is critical to indicate missing information rather than provide incorrect data.

## 4. Output Requirements

- Your SOLE output must be the fully updated content of the `./cursor/rules/_user-<username>.mdc` file.
- Ensure the output is valid Markdown.

---
**REMEMBER: Your objective is to produce an accurate, detailed, and comprehensive `./cursor/rules/_user-<username>.mdc` file. This file MUST be populated using *only* information extracted directly from THIS project's codebase and its documentation. Adhere strictly to the provided template. After updating the file with all available project-specific information (or marking details as 'not found'), your task is complete.**
---

## .cursor/rules/_user-<username>.mdc Template

Use this template as guide for creating the _user-<username>.mdc:

<template>
---
description:
globs:
alwaysApply: true
---

# [Project Name] - Project Rules

## 1. Project Overview

goal: [Concisely describe the main purpose and goal of the project. What problem does it solve? Source from README or project vision docs.]
type: [e.g., Web Application, CLI Tool, Library, Mobile App, Backend API, Monorepo. Determine from project structure and build files.]
key features:
- [core functionality or feature 1, derived from docs or main modules]
- [core functionality or feature 2, derived from docs or main modules]
- [core functionality or feature 3, derived from docs or main modules]

## 2. Technology Stack


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skillrecordings) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
