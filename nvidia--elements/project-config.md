---
trigger: always_on
description: This file provides guidance to AI Agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI Agents when working with code in this repository.

## Instruction Precedence for Agents

When instructions conflict, follow this order (highest to lowest):

1. Runtime or platform constraints from the agent host
2. Repository agent rules in `AGENTS.md`
3. Task-specific skill instructions in `.claude/skills/*/SKILL.md`
4. Documents explicitly required by a selected skill (for example, docs in `projects/site/src/docs/internal/guidelines/`)
5. General project documentation such as `README.md`

Notes:

- `README.md` is primarily for orientation and setup; it does not override agent policy.
- If a skill says you MUST review a referenced guideline, treat that guideline as required for that task.
- If two same-level sources conflict, prefer the more specific and recently maintained source, and note the assumption in your response.

## Repository Overview

Elements is a design language for AI/ML factories built as a monorepo containing framework agnostic Web Components (Lit), themes, styles, testing utilities, and starter templates. The repository uses pnpm workspaces with Wireit for build orchestration and Semantic Release for automated versioning/publishing.

## Environment Requirements

- **nvm**: Node Version Manager for managing Node.js versions
- **Node.js**: 26.1.0 (enforced via `.nvmrc` and `package.json` engines)
- **pnpm**: 11.0.8 (managed via Corepack 0.34.7)
- **Git LFS**: Required for visual test screenshots and videos (`.gitattributes` defines tracked files)
- **Playwright**: Browser-based testing uses Chromium (installed via prepare script)
- **Vale**: Prose linter for documentation and JSDoc (installed via prepare script)

## Common Commands

### Repository Setup

```shell
# Install git-lfs if not already installed
brew install git-lfs
git lfs install
git lfs pull

# Install nvm if not already installed
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
. ~/.nvm/nvm.sh

# Install dependencies
nvm install
npm install -g corepack@0.34.7
corepack enable
corepack prepare --activate
pnpm i --frozen-lockfile --prefer-offline
```

### Building and Testing

```shell
# Run full CI pipeline locally (lint, build, test)
pnpm run ci

# Run all tests and lighthouse tests
pnpm run ci:all

# Clean and reset repository
pnpm run ci:reset

# Format code
pnpm run format
pnpm run format:fix

# Prose lint (documentation and JSDoc)
pnpm run lint:vale
```

### Individual Project Commands

Each project has a `DEVELOPMENT.md` file with the authoritative list of available pnpm scripts for that project. Consult `projects/<name>/DEVELOPMENT.md` before running commands. The common commands below are a quick reference. Run them from within the specific project directory (for example, `cd projects/core`):

```shell
# Development watch mode
pnpm run dev

# Build project
pnpm run build

# Run unit tests
pnpm run test

# Run single test suite
pnpm run test -- src/badge/badge.test.ts

# Run accessibility tests
pnpm run test:axe

# Run lighthouse performance tests
pnpm run test:lighthouse

# Run visual regression tests
pnpm run test:visual

# Run SSR tests
pnpm run test:ssr

# Lint project
pnpm run lint
pnpm run lint:fix
```

## Architecture

### Monorepo Structure

The repository contains a top-level workspace with individual project directories:

- `/projects/core` - Core Elements Web Components library (Lit-based)
- `/projects/forms` - Form control utilities, mixins, and schema validation
- `/projects/themes` - Theme tokens and CSS custom properties
- `/projects/styles` - CSS utilities for layout and typography
- `/projects/starters` - Starter templates for supported frameworks, including React, Angular, Vue, and Svelte
- `/projects/cli` - Command-line tooling for Elements development and project scaffolding
- `/projects/create` - `npm create @nvidia-elements` wrapper around CLI project creation
- `/projects/code` - Code authoring components, including syntax-highlighted code blocks
- `/projects/markdown` - Markdown components and utilities
- `/projects/media` - Media playback UI components
- `/projects/lint` - Elements lint configurations and custom rules
- `/projects/monaco` - Monaco editor integration
- `/projects/pages` - GitHub Pages deployment project
- `/projects/site` - Documentation site (11ty)
- `/projects/internals` - Internal tooling (vite configs, eslint, patterns, metadata)

### Component Architecture

Each component in `/projects/core/src/` follows this structure:

```
component-name/
├── component-name.ts                 # Main component class (extends LitElement or applies a forms mixin)
├── component-name.css                # Component styles
├── component-name.examples.ts        # Example templates for documentation
├── component-name.test.ts            # Unit tests
├── component-name.test.axe.ts        # Accessibility tests
├── component-name.test.lighthouse.ts # Performance tests
├── component-name.test.visual.ts     # Visual regression tests
├── component-name.test.ssr.ts        # SSR tests
├── index.ts                          # Exports component class (no side effects)
└── define.ts                         # Registers component to customElementsRegistry
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/elements](https://github.com/NVIDIA/elements) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
