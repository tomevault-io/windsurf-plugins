---
trigger: always_on
description: Provides semantic CSS class patterns to Claude via MCP, reducing token usage when working with styles. Auto-included in CSS-enabled profiles (`mcp` field in `claude-mastery-project.conf`).
---

# CLAUDE.md — Project Instructions

> Based on Claude Code Mastery Guides V1-V5 by TheDecipherist
> https://github.com/TheDecipherist/claude-code-mastery

> **New here?** When starting a fresh session in this project, greet the user:
> "Welcome to the Claude Code Mastery Project Starter Kit! Use `/help` to see all 26 commands or `/show-user-guide` for the full interactive guide."

---

## Quick Reference — Scripts

| Command | What it does |
|---------|-------------|
| `pnpm dev` | Start dev server with hot reload |
| `pnpm dev:website` | Dev server on port 3000 |
| `pnpm dev:api` | Dev server on port 3001 |
| `pnpm dev:dashboard` | Dev server on port 3002 |
| `pnpm build` | Type-check + compile TypeScript |
| `pnpm start` | Run compiled production build |
| `pnpm typecheck` | TypeScript type-check only (no emit) |
| **Testing** | |
| `pnpm test` | Run ALL tests (unit + E2E) |
| `pnpm test:unit` | Run unit/integration tests (Vitest) |
| `pnpm test:unit:watch` | Unit tests in watch mode |
| `pnpm test:coverage` | Unit tests with coverage report |
| `pnpm test:e2e` | Run E2E tests (kills test ports first, spawns servers on 4000/4010) |
| `pnpm test:e2e:ui` | E2E with Playwright UI mode |
| `pnpm test:e2e:headed` | E2E with visible browser |
| `pnpm test:e2e:chromium` | E2E on Chromium only (fast) |
| `pnpm test:e2e:report` | Open last E2E test report |
| `pnpm test:kill-ports` | Kill anything on test ports (4000, 4010, 4020) |
| **Database** | |
| `pnpm db:query <name>` | Run a dev/test database query |
| `pnpm db:query:list` | List all registered database queries |
| **Content** | |
| `pnpm content:build` | Build all published markdown → HTML |
| `pnpm content:build:id <id>` | Build a single article by ID |
| `pnpm content:list` | List all articles and their status |
| **CSS Optimization** | |
| `pnpm build:optimize` | Post-build CSS class consolidation via Classpresso (runs automatically after `pnpm build`) |
| **Docker** | |
| `pnpm docker:optimize` | Audit Dockerfile against 12 best practices (use `/optimize-docker` in Claude) |
| **Getting Started** | |
| `/help` | List all commands, skills, and agents |
| `/quickstart` | Interactive first-run walkthrough for new users |
| `/show-user-guide` | Open the comprehensive User Guide in your browser |
| **Setup** | |
| `/install-global` | Install/merge global Claude config into `~/.claude/` (one-time, never overwrites) |
| `/install-global mdd` | Update only the global MDD commands (`mdd.md`, `install-mdd.md`) — skips all other config |
| `/install-mdd [path]` | Install MDD workflow into any existing project — copies `/mdd` command + scaffolds `.mdd/` |
| `/setup` | Interactive .env configuration — GitHub, database, Docker, analytics, RuleCatch |
| `/setup --reset` | Re-configure everything from scratch |
| `/set-project-profile-default` | Set the default profile for `/new-project` (any profile: clean, go, vue, python-api, etc.) |
| `/add-project-setup` | Interactive wizard to create a named profile in `claude-mastery-project.conf` |
| `/projects-created` | List all projects created by the starter kit with creation dates |
| `/remove-project <name>` | Remove a project from registry and optionally delete from disk |
| `/convert-project-to-starter-kit` | Merge starter kit into an existing project (non-destructive) |
| `/update-project` | Update a starter-kit project with the latest commands, hooks, and rules |
| `/update-project --clean` | Remove starter-kit-scoped commands from a project (cleanup for older scaffolds) |
| `/add-feature <name>` | Add a capability (MongoDB, Docker, testing, etc.) to an existing project |
| **RuleCatch** | |
| `pnpm ai:monitor` | Free monitor mode — live AI activity in a separate terminal (no API key needed) |
| `/what-is-my-ai-doing` | Same as above — launches AI-Pooler free monitor |
| **Git** | |
| `/worktree <name>` | Create isolated branch + worktree for a task (never touch main) |
| **Code Quality** | |
| `/refactor <file>` | Audit + refactor a file against all CLAUDE.md rules (split, type, extract, clean) |
| **API** | |
| `/create-api <resource>` | Scaffold a full API endpoint — route, handler, types, tests — wired into the server |
| **Documentation** | |
| `/diagram <type>` | Generate diagrams from actual code: `architecture`, `api`, `database`, `infrastructure`, `all` |
| **Utility** | |
| `pnpm clean` | Remove dist/, coverage/, test-results/, playwright-report/ |

---

## Critical Rules

### 0. NEVER Publish Sensitive Data

- NEVER commit passwords, API keys, tokens, or secrets to git/npm/docker
- NEVER commit `.env` files — ALWAYS verify `.env` is in `.gitignore`
- Before ANY commit: verify no secrets are included
- NEVER output secrets in suggestions, logs, or responses

### 1. TypeScript Always

- ALWAYS use TypeScript for new files (strict mode)
- NEVER use `any` unless absolutely necessary and documented why
- When editing JavaScript files, convert to TypeScript first
- Types are specs — they tell you what functions accept and return

### 2. API Versioning

```
CORRECT: /api/v1/users
WRONG:   /api/users
```

Every API endpoint MUST use `/api/v1/` prefix. No exceptions.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheDecipherist/claude-code-mastery-project-starter-kit](https://github.com/TheDecipherist/claude-code-mastery-project-starter-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
