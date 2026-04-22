---
trigger: always_on
description: This document defines the development standards for the Universal Development Standards project itself. As a framework that provides development standards to other projects, we practice what we preach ("dogfooding").
---

# Universal Development Standards - Project Guidelines

This document defines the development standards for the Universal Development Standards project itself. As a framework that provides development standards to other projects, we practice what we preach ("dogfooding").

## Project Overview

Universal Development Standards is a language-agnostic, framework-agnostic development standards framework. It provides:

- **Core Standards** (`core/`): 70+ fundamental development standards
- **AI Skills** (`skills/`): Claude Code skills for AI-assisted development
- **CLI Tool** (`cli/`): Node.js CLI for adopting standards
- **Integrations** (`integrations/`): Configurations for various AI tools
- **Localization** (`locales/`): Multi-language support (English, Traditional Chinese)

## Tech Stack

| Component | Technology | Version |
|-----------|------------|---------|
| Runtime | Node.js | >= 18.0.0 |
| Module System | ES Modules | - |
| Testing | Vitest | ^4.0.16 |
| Linting | ESLint | ^8.56.0 |
| CLI Framework | Commander.js | ^12.1.0 |
| Interactive Prompts | Inquirer.js | ^9.2.12 |

## Development Standards

> **Core Standards Usage Rule**:
> When verifying standards, checking code, or performing tasks, **PRIORITIZE** reading the concise rules in `core/` (e.g., `core/testing-standards.md`).
> **ONLY** read `core/guides/` or `methodologies/guides/` when explicitly asked for educational content, detailed explanations, or tutorials.
> This ensures token efficiency and focused context.

### 1. Commit Message Format

Follow the Conventional Commits specification defined in `core/commit-message-guide.md`:

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature or standard
- `fix`: Bug fix or error correction
- `docs`: Documentation updates
- `chore`: Maintenance tasks
- `test`: Test-related changes
- `refactor`: Code refactoring
- `style`: Formatting changes

**Examples:**
```bash
feat(core): add new testing completeness dimensions
docs(skills): update Claude Code skill documentation
fix(cli): resolve path resolution issue on Windows
chore(i18n): sync translations with source files
```

### 2. Branch Strategy

Follow the Git workflow defined in `core/git-workflow.md`:

| Branch | Purpose |
|--------|---------|
| `main` | Stable, production-ready releases |
| `feature/*` | New features and enhancements |
| `fix/*` | Bug fixes |
| `docs/*` | Documentation updates |
| `chore/*` | Maintenance tasks |

### 3. Code Style

**JavaScript:**
- Use single quotes for strings
- End statements with semicolons
- Use ES Module syntax (`import`/`export`)
- Follow ESLint configuration in `cli/.eslintrc.json`

**Markdown:**
- Use ATX-style headers (`#`, `##`, `###`)
- Include blank lines before and after headers
- Use fenced code blocks with language specification

### 4. Testing Requirements

- All CLI features must have corresponding tests
- Run tests before committing: `npm test` (in `cli/` directory)
- Run linting: `npm run lint` (in `cli/` directory)
- Test coverage reports: `npm run test:coverage`

### 5. Translation Sync

When modifying core standards:
1. Update the English source file first
2. Sync changes to `locales/zh-TW/` directory
3. Run translation check: `./scripts/check-translation-sync.sh`

## Quick Commands

### macOS / Linux

```bash
# CLI development (run from cli/ directory)
cd cli
npm install          # Install dependencies
npm test             # Run tests
npm run test:watch   # Run tests in watch mode
npm run lint         # Check code style
npm run test:coverage # Generate coverage report

# Translation sync check (run from root)
./scripts/check-translation-sync.sh

# Version sync check (run from root)
./scripts/check-version-sync.sh

# Standards consistency check (run from root)
./scripts/check-standards-sync.sh

# Pre-release check (run all checks at once)
./scripts/pre-release-check.sh

# Local CLI testing
node cli/bin/uds.js list
node cli/bin/uds.js init --help

# Health score (self-diagnosis)
node cli/bin/uds.js audit --score --self
node cli/bin/uds.js audit --score --self --format json

# External reference check
node scripts/check-external-references.mjs

# Hook statistics analysis (requires opt-in: .uds/config.json {"hookStats": true})
node scripts/analyze-hook-stats.mjs
```

### Windows (PowerShell)

```powershell
# CLI development (run from cli\ directory)
cd cli
npm install          # Install dependencies
npm test             # Run tests
npm run test:watch   # Run tests in watch mode
npm run lint         # Check code style
npm run test:coverage # Generate coverage report

# Translation sync check (run from root)
.\scripts\check-translation-sync.ps1

# Version sync check (run from root)
.\scripts\check-version-sync.ps1

# Standards consistency check (run from root)
.\scripts\check-standards-sync.ps1

# Pre-release check (run all checks at once)
.\scripts\pre-release-check.ps1

# Local CLI testing
node cli\bin\uds.js list
node cli\bin\uds.js init --help
```

### Windows (Git Bash)

```bash
# Same commands as macOS / Linux work in Git Bash
./scripts/check-translation-sync.sh
./scripts/check-version-sync.sh

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AsiaOstrich/universal-dev-standards](https://github.com/AsiaOstrich/universal-dev-standards) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
