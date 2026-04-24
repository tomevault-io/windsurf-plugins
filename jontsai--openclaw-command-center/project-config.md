---
trigger: always_on
description: > _"The Overmind speaks through many voices, but with one purpose."_
---

# AGENTS.md — AI Workspace Guide

> _"The Overmind speaks through many voices, but with one purpose."_

Welcome, AI agent. This document defines how you should interact with this codebase.

## ⚠️ CRITICAL: Pull Request Workflow

**All changes to this repository MUST go through pull requests.**

This is a public open source project. Even maintainers (including AI agents working on behalf of maintainers) must:

1. Create a feature branch (`git checkout -b type/description`)
2. Make changes and commit
3. Push branch and open a PR
4. Get approval before merging

**Never push directly to `main`.** This applies to everyone, including the repo owner.

## 🎯 Mission

OpenClaw Command Center is the central dashboard for AI assistant management. Your mission is to help build, maintain, and improve this system while maintaining the Starcraft/Zerg thematic elements that make it unique.

## 🏛️ Architecture

**Read First**: [`docs/architecture/OVERVIEW.md`](docs/architecture/OVERVIEW.md)

Key architectural principles:

1. **DRY** — Don't Repeat Yourself. Extract shared code to partials/modules.
2. **Zero Build Step** — Plain HTML/CSS/JS, no compilation needed.
3. **Real-Time First** — SSE for live updates, polling as fallback.
4. **Progressive Enhancement** — Works without JS, enhanced with JS.

## 📁 Workspace Structure

```
openclaw-command-center/
├── lib/                    # Core server logic
│   ├── server.js           # Main HTTP server and API routes
│   ├── config.js           # Configuration loader with auto-detection
│   ├── jobs.js             # Jobs/scheduler API integration
│   ├── linear-sync.js      # Linear issue tracker integration
│   └── topic-classifier.js # NLP-based topic classification
├── public/                 # Frontend assets
│   ├── index.html          # Main dashboard UI
│   ├── jobs.html           # AI Jobs management UI
│   ├── partials/           # ⭐ Shared HTML partials (DRY!)
│   │   └── sidebar.html    # Navigation sidebar component
│   ├── css/
│   │   └── dashboard.css   # Shared styles
│   └── js/
│       ├── sidebar.js      # Sidebar loader + SSE badges
│       ├── app.js          # Main dashboard logic
│       └── lib/            # Third-party libraries
├── scripts/                # Operational scripts
├── config/                 # Configuration (be careful!)
├── docs/                   # Documentation
│   └── architecture/       # Architecture Decision Records
├── tests/                  # Test files
├── SKILL.md                # ClawHub skill metadata
└── package.json            # Version and dependencies
```

## ✅ Safe Operations

Do freely:

- Read any file to understand the codebase
- Create/modify files in `lib/`, `public/`, `docs/`, `tests/`
- Add tests
- Update documentation
- Create feature branches

## ⚠️ Ask First

Check with a human before:

- Modifying `config/` files
- Changing CI/CD workflows
- Adding new dependencies to `package.json`
- Making breaking API changes
- Anything touching authentication/secrets

## 🚫 Never

- **Push directly to `main` branch** — ALL changes require PRs
- Commit secrets, API keys, or credentials
- Commit user-specific data files (see `public/data/AGENTS.md`)
- Delete files without confirmation
- Expose internal endpoints publicly

## 🛠️ Development Workflow

### 0. First-Time Setup

```bash
# Install pre-commit hooks (required for all contributors)
make install-hooks

# Or manually:
cp scripts/pre-commit .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

The pre-commit hook enforces rules from this file automatically.

### 1. Feature Development

```bash
# Create feature branch
git checkout -b feat/your-feature-name

# Make changes, then test locally
npm test
npm run lint
make check  # Run pre-commit checks manually

# Commit with descriptive message
git commit -m "feat: add overlord status indicator"

# Push and create PR
git push -u origin feat/your-feature-name
```

### 2. Commit Message Convention

Follow [Conventional Commits](https://www.conventionalcommits.org/):

- `feat:` — New feature
- `fix:` — Bug fix
- `docs:` — Documentation only
- `style:` — Formatting, no code change
- `refactor:` — Code restructuring
- `test:` — Adding tests
- `chore:` — Maintenance tasks

### 3. Code Style

- Use ESLint configuration provided
- Prettier for formatting
- JSDoc comments for public functions
- Meaningful variable names (thematic names encouraged!)

## 📦 ClawHub Skill Workflow

This project is distributed as a ClawHub skill. After changes are merged to `main`, they need to be published to the registry so users can install/update via `clawhub install command-center`.

### Understanding Skill Metadata

Two files control the skill identity:

- **`SKILL.md`** — Frontmatter (`name`, `version`, `description`) used by ClawHub for discovery and search
- **`package.json`** — `version` field for npm compatibility

⚠️ **CRITICAL: Version Sync Required**

Both `package.json` and `SKILL.md` **MUST have the same version number**. This is enforced by pre-commit hooks.

```bash
# If you change version in one file, change it in both:
# package.json:  "version": "1.0.4"
# SKILL.md:      version: 1.0.4
```

The pre-commit hook will block commits if versions are out of sync.

### Publishing Updates

```bash
# 1. Authenticate (one-time)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jontsai/openclaw-command-center](https://github.com/jontsai/openclaw-command-center) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
