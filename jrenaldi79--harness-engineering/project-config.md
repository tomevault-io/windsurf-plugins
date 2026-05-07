---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

**harness-engineering** is a Claude Code plugin and reference for AI coding agent harnesses. It provides two skills (`/readiness` and `/setup`) that analyze and configure projects for agent-assisted development, plus a README mapping 20+ best practices from industry sources.

### Core Features

- **`/readiness`**: Scores a codebase across 8 pillars and 5 maturity levels, produces a saved report with delta tracking
- **`/setup`**: Scaffolds CLAUDE.md files, enforcement scripts, git hooks, linter configs, and agent settings via Socratic questioning
- **Reference guide**: Maps best practices from OpenAI, Anthropic, Augment Code, Factory.ai, and practitioners to concrete implementation patterns

---

## Essential Commands

### Testing
```bash
node --experimental-vm-modules node_modules/.bin/jest tests/scripts/   # Unit tests for setup scripts
bash tests/evals/run-evals.sh                                          # E2E readiness evals (default)
bash tests/evals/run-evals.sh --config setup-eval-config.json          # E2E setup evals
bash tests/evals/test-marketplace-install.sh                           # Test plugin install flow
```

### Validation
```bash
node skills/setup/scripts/lib/generate-docs.js --check   # Verify auto-generated sections are current
node skills/setup/scripts/lib/validate-docs.js --full     # Check for documentation drift
```

### Setup
```bash
bash scripts/install-hooks.sh   # Install git hooks (pre-commit + pre-push)
```

---

## Architecture

<!-- AUTO:tree -->
skills/
├── readiness/
│   └── SKILL.md
└── setup/
    ├── references/
    │   ├── claude-md-guide.md
    │   ├── enforcement-scripts.md
    │   └── stack-node-typescript.md
    ├── scripts/
    │   ├── hooks/
    │   │   ├── pre-commit
    │   │   └── pre-push
    │   ├── lib/
    │   │   ├── check-file-sizes.js  # File size enforcement script for pre-commit hook.
    │   │   ├── check-secrets.js  # Secret detection script for pre-commit hook.
    │   │   ├── check-test-colocation.js  # Test colocation enforcement script for pre-commit hook.
    │   │   ├── generate-docs-helpers.js  # Helper functions for generate-docs.js.
    │   │   ├── generate-docs.js  # Auto-generate CLAUDE.md sections from source code.
    │   │   └── validate-docs.js  # CLAUDE.md drift detection script.
    │   ├── generate-claude-md.js  # Generate tailored CLAUDE.md files for a project from templates.
    │   ├── init-project.js  # Project scaffolding script for Node/TypeScript projects.
    │   └── install-enforcement.js  # Copies enforcement tooling into a target project.
    ├── templates/
    │   ├── rules/
    │   │   ├── code-quality.md
    │   │   ├── react.md
    │   │   ├── tdd.md
    │   │   ├── testing.md
    │   │   └── typescript.md
    │   ├── eslint-base.js
    │   ├── gitignore-template
    │   ├── global-claude.md
    │   ├── lint-staged.config.js
    │   ├── project-claude.md
    │   └── settings.json
    └── SKILL.md
scripts/
├── hooks/
│   ├── pre-commit
│   └── pre-push
├── install-hooks.sh
├── README.md
├── release.sh
└── repo-generate-docs.js  # Repo-level CLAUDE.md auto-generator.
tests/
├── evals/
│   ├── eval-config.json
│   ├── grader.js  # Readiness Skill Grader
│   ├── hook-commit-validator.js  # Hook-driven commit validation for setup eval grader.
│   ├── README.md
│   ├── run-evals.sh
│   ├── setup-eval-config.json
│   ├── setup-grader.js  # Setup Skill Grader — validates /setup output against setup-eval-config.json.
│   ├── setup-readiness-eval-config.json
│   ├── setup-readiness-grader.js  # Setup-then-Readiness Grader — validates that /setup produces a project
│   └── test-marketplace-install.sh
└── scripts/
    ├── check-file-sizes.test.js  # Tests for skills/setup/scripts/lib/check-file-sizes.js
    ├── check-secrets.test.js  # Tests for skills/setup/scripts/lib/check-secrets.js
    ├── check-test-colocation.test.js  # Tests for skills/setup/scripts/lib/check-test-colocation.js
    ├── detect-source-dirs.test.js  # Tests for detectSourceDirs and buildModuleIndex adaptive scanning.
    ├── doc-drift-detection.test.js  # Tests for doc drift detection — verifies validate-docs.js catches
    ├── doc-indexing-roundtrip.test.js  # Tests for post-install doc indexing round-trip — verifies that
    ├── enforcement-roundtrip.test.js  # Tests for post-install enforcement script round-trip — verifies that scripts
    ├── generate-claude-md.test.js  # Tests for skills/setup/scripts/generate-claude-md.js
    ├── generate-docs-helpers.test.js  # Tests for generate-docs-helpers.js: directory trees, module indexes,
    ├── generate-docs.test.js  # Tests for generate-docs.js marker operations: replaceMarkers,
    ├── hook-integration.test.js  # Tests for git commit hook integration — verifies that git commit triggers
    ├── incremental-doc-indexing.test.js  # Tests for incremental doc indexing — verifies that adding a new source file
    ├── init-project.test.js  # Tests for skills/setup/scripts/init-project.js
    ├── install-enforcement.test.js  # Tests for skills/setup/scripts/install-enforcement.js

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jrenaldi79/harness-engineering](https://github.com/jrenaldi79/harness-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
