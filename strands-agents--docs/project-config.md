---
trigger: always_on
description: This document provides guidance specifically for AI agents working on the strands-agents/private-docs-staging codebase. For human contributor guidelines, see [CONTRIBUTING.md](CONTRIBUTING.md).
---

# Agent Development Guide - strands-agents/private-docs-staging

This document provides guidance specifically for AI agents working on the strands-agents/private-docs-staging codebase. For human contributor guidelines, see [CONTRIBUTING.md](CONTRIBUTING.md).

## Purpose and Scope
The goal of this repository is to revamp this documentation repo so that it provides clear and well organized documentation on how to develop with Strands SDK with either Python or Typescript.

**AGENTS.md** contains agent-specific repository information including:
- Directory structure with summaries of what is included in each directory
- Development workflow instructions for agents to follow when developing features
- Coding patterns and testing patterns to follow when writing code
- Style guidelines, organizational patterns, and best practices

**For human contributors**: See [CONTRIBUTING.md](CONTRIBUTING.md) for setup, testing, and contribution guidelines.

## Team Process Documents

When working on SDK features or documentation, familiarize yourself with these team processes:

* **[Feature Lifecycle Process](team/FEATURE_LIFECYCLE.md)**: How features are added, versioned, deprecated, and graduated from experimental status
* **[API Bar Raising](team/API_BAR_RAISING.md)**: Standards for API design quality
* **[Decisions](team/DECISIONS.md)**: Key architectural and design decisions
* **[Tenets](team/TENETS.md)**: Core principles guiding SDK development

These documents define the standards and processes that ensure consistency and quality across the Strands SDK.

## Directory Structure

```
├── AGENTS.md
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── SITE-ARCHITECTURE.md          # Detailed Astro/Starlight customizations
├── src/                          # Astro source files
│   ├── components/               # Custom Astro components
│   │   ├── overrides/            # Starlight component overrides
│   │   └── ...
│   ├── config/                   # Site configuration
│   ├── content/                  # Content collections
│   │   └── docs/                 # Documentation content (Markdown/MDX)
│   │       ├── api/
│   │       │   ├── python/
│   │       │   │   └── _generated/   # Symlink to .build/api-docs/python
│   │       │   └── typescript/
│   │       │       └── _generated/   # Symlink to .build/api-docs/typescript
│   │       ├── assets/
│   │       ├── community/
│   │       ├── contribute/
│   │       ├── examples/
│   │       ├── labs/
│   │       └── user-guide/
│   ├── layouts/                  # Custom layouts
│   ├── pages/                    # Astro pages
│   ├── plugins/                  # Remark/Rehype plugins
│   ├── styles/                   # Global styles
│   └── util/                     # Utility functions
├── mkdocs.yml                    # Navigation structure (still used by Astro)
├── astro.config.mjs              # Astro configuration
├── package.json                  # Node.js dependencies and scripts
├── tsconfig.json                 # TypeScript configuration
├── LICENSE
├── NOTICE
├── README.md
├── overrides/                    # Legacy MkDocs overrides (being migrated)
├── scripts/                      # Build and utility scripts
├── test/                         # Test files
└── test-snippets/                # TypeScript snippet test files
```
### Directory Purposes


**IMPORTANT**: After making changes that affect the directory structure (adding new directories, moving files, or adding significant new files), you MUST update this directory structure section to reflect the current state of the repository.

## Development Workflow for Agents

### 1. Environment Setup
#### Prerequisites

- Python 3.10+
- Node.js 20+, npm

#### Setup and Installation

```bash
npm install
```

#### Building and Previewing

Generate the static site:

```bash
npm run build
```

Run a local development server at http://localhost:4321/:

```bash
npm run dev
```

### 2. Making Changes

1. **Create feature branch**: `git checkout -b agent-tasks/{ISSUE_NUMBER}`
2. **Implement changes** following the patterns below
3. **Run quality checks** before committing (pre-commit hooks will run automatically)
4. **Commit with conventional commits**: `feat:`, `fix:`, `refactor:`, `docs:`, etc.
5. **Push to remote**: `git push origin agent-tasks/{ISSUE_NUMBER}`

### 3. Quality Gates

Pre-commit hooks automatically run:
- Unit tests (via npm test)
- Format checking (via npm run format:check)
- Type checking (via npm run typecheck)

All checks must pass before commit is allowed.

## Coding Patterns and Best Practices

### Code Style Guidelines (for Typescript)

**Formatting** (enforced by Prettier):
- No semicolons
- Single quotes
- Line length: 120 characters
- Line length for doc snippet files under `src/content/docs/`: 90 characters
- Tab width: 2 spaces
- Trailing commas in ES5 style
- Template literal contents in doc snippets must also stay under 90 characters per line. Prettier does not enforce this automatically.

**Example**:
```typescript
export function example(name: string, options?: Options): Result {
  const config = {
    name,
    enabled: true,
    settings: {
      timeout: 5000,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [strands-agents/docs](https://github.com/strands-agents/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
