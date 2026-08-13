---
trigger: always_on
description: This document provides guidelines for AI agents developing prisma-strong-migrations.
---

# AGENTS.md - AI Agent Development Guidelines

This document provides guidelines for AI agents developing prisma-strong-migrations.

## Project Overview

prisma-strong-migrations is a CLI tool that detects dangerous operations in Prisma migrations.

## Development Environment

### Docker / devcontainer (Recommended)

This project uses Docker + devcontainer for development.

```bash
# Open devcontainer in VSCode
# 1. Open the project in VSCode
# 2. Command Palette (Cmd+Shift+P) → "Dev Containers: Reopen in Container"
```

The devcontainer includes:

- Node.js v25
- Vite+ (vp command)
- Git

### Environment Configuration Files

```
.devcontainer/
├── devcontainer.json  # devcontainer configuration
├── Dockerfile         # Container image definition
└── post-create.sh     # Initialization script
```

### Key Documentation

| Document                               | Content                                              |
| -------------------------------------- | ---------------------------------------------------- |
| [README.md](./README.md)               | Project overview, Bad/Good examples for all rules    |
| [docs/DESIGN.md](./docs/DESIGN.md)     | Architecture, directory structure, component details |
| [docs/RULES.md](./docs/RULES.md)       | Technical details of rules                           |
| [docs/TESTING.md](./docs/TESTING.md)   | Testing strategy                                     |
| [docs/WORKFLOW.md](./docs/WORKFLOW.md) | Development workflow                                 |

## Development Rules

### 1. Commit Rules (Most Important)

**Commit frequently with small changes.**

```bash
# ✅ Good: Small, focused commits
git add src/parser/sql-parser.ts
git commit -m "feat(parser): add SQL parser base implementation"

git add src/parser/sql-parser.test.ts
git commit -m "test(parser): add SQL parser unit tests"

# ❌ Bad: Large changes in a single commit
git add .
git commit -m "add parser and tests and rules"
```

#### When to Commit

- After implementing a single function → commit
- After adding tests → commit
- After fixing a bug → commit
- After updating documentation → commit

#### Installing New Libraries

Always separate library installation from the changes that use it.

```bash
# ✅ Good: Installation and fixes in separate commits
git add package.json pnpm-lock.yaml knip.json
git commit -m "chore: add knip for unused code detection"

git add src/...
git commit -m "fix: resolve knip findings"

# ❌ Bad: Installation and fixes in the same commit
git add .
git commit -m "add knip and fix unused exports"
```

#### Commit Message Format

```
<type>(<scope>): <description>

# Examples:
feat(parser): add SQL parser for ALTER TABLE statements
test(rules): add remove-column rule tests
fix(cli): handle missing migration directory
docs(readme): add installation instructions
refactor(checker): extract common validation logic
```

### 2. Directory Structure

```
prisma-strong-migrations/
├── src/
│   ├── index.ts               # Main exports
│   ├── cli.ts                 # CLI entry point
│   ├── checker.ts             # Check execution engine
│   ├── checker.test.ts        # ← Tests in same directory
│   │
│   ├── parser/
│   │   ├── index.ts
│   │   ├── sql-parser.ts
│   │   ├── sql-parser.test.ts # ← Tests in same directory
│   │   └── types.ts
│   │
│   ├── rules/
│   │   ├── index.ts
│   │   ├── types.ts
│   │   ├── loader.ts
│   │   └── builtin/           # Each rule in separate file
│   │       ├── remove-column.ts
│   │       ├── remove-column.test.ts
│   │       └── ...
│   │
│   ├── reporter/
│   │   ├── console-reporter.ts
│   │   └── json-reporter.ts
│   │
│   └── config/
│       └── types.ts
│
├── integration-tests/
│   ├── cases/                 # Test cases (SQL files + expected results)
│   │   ├── remove-column/
│   │   │   ├── migration.sql
│   │   │   ├── expected.json
│   │   │   └── README.md
│   │   └── ...
│   └── run-tests.ts
│
└── vite.config.ts             # Vite+ configuration
```

### 3. Development Toolchain

**Use Vite+.**

```bash
# Install dependencies
vp install

# Run tests
vp test

# Lint, format, and type check
vp check

# Build library
vp pack
```

#### Running CLI Tools

**All CLI tools must be installed as `devDependencies` and executed via `vp exec`.**

```bash
# ✅ Good: install as devDependency, run via vp exec
vp add -D knip
vp exec knip

# ❌ Bad: run via npx (downloads on the fly, not reproducible)
npx knip
```

- `vp exec <tool>` — runs a binary from `node_modules/.bin` (equivalent to `pnpm exec`)
- `vp dlx <tool>` — runs without installing (use only for one-off exploration, not in CI or scripts)

### 4. Implementation Order

Follow this order for implementation:

#### Phase 1: Project Initialization

```bash
# 1. Create package.json
# 2. Create vite.config.ts
# 3. Create tsconfig.json
# 4. Install dependencies
```

**Commit example:**

```
feat: initialize project with vite-plus
```

#### Phase 2: Type Definitions

```bash
# 1. src/parser/types.ts - ParsedStatement type
# 2. src/rules/types.ts - Rule type, CheckContext type
# 3. src/config/types.ts - Config type
```

**Commit examples:**

```
feat(types): add ParsedStatement type
feat(types): add Rule and CheckContext types
feat(types): add Config type
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [soartec-lab/prisma-strong-migrations](https://github.com/soartec-lab/prisma-strong-migrations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
