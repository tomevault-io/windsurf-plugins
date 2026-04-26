---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Core Development Principles

1. **Proper Solutions Over Quick Fixes** - Implement correctly the first time
2. **Root Cause Analysis** - Fix underlying issues, not symptoms
3. **Stability Over Speed** - This is a production template
4. **Clean Architecture** - Follow established patterns consistently
5. **No Technical Debt** - Never commit TODOs or workarounds

## Docker-First Development (MANDATORY)

**CRITICAL**: This project REQUIRES Docker. Local pnpm/npm commands are NOT supported.

### NEVER Install Packages Locally

**ABSOLUTELY FORBIDDEN** - Never run these commands on the host machine:

```bash
# ❌ CRITICAL NO - NEVER do any of these locally
npm install
npm install --no-save <package>
pnpm install
pnpm add <package>
yarn install
npx <anything>

# ✅ CORRECT - Always use Docker
docker compose exec hogball pnpm install
docker compose exec hogball pnpm add <package>
```

**Why this is critical:**

- Creates local `node_modules` with wrong permissions (Docker-owned)
- Causes conflicts between host and container dependencies
- Breaks the Docker-first architecture
- Creates cleanup nightmares (Docker-owned files can't be deleted by host user)

**If you accidentally installed locally:**

```bash
docker compose down
docker compose run --rm hogball rm -rf node_modules
docker compose up
```

### NEVER Use sudo - Use Docker Instead

When encountering permission errors, **NEVER use `sudo`**. Use Docker:

```bash
# ❌ WRONG - Don't do this
sudo chown -R $USER:$USER .next
sudo rm -rf node_modules

# ✅ CORRECT - Use Docker
docker compose exec hogball rm -rf .next
docker compose exec hogball rm -rf node_modules
docker compose down && docker compose up
```

**Why**: The container runs as your user (UID/GID from .env). Docker commands execute with correct permissions automatically.

**Permission errors? Always try:**

1. `docker compose down && docker compose up` (restarts container, cleans .next)
2. `docker compose exec hogball pnpm run docker:clean`

### Essential Commands

```bash
# Start development
docker compose up

# Development server
docker compose exec hogball pnpm run dev

# Run tests
docker compose exec hogball pnpm test
docker compose exec hogball pnpm run test:suite    # Full suite

# Storybook
docker compose exec hogball pnpm run storybook

# E2E tests
docker compose exec hogball pnpm exec playwright test

# Type checking & linting
docker compose exec hogball pnpm run type-check
docker compose exec hogball pnpm run lint

# Clean start if issues
docker compose exec hogball pnpm run docker:clean
```

### Git Commits from Docker

Git hooks may fail when running locally if the repo was set up inside Docker. Always commit from inside the container:

```bash
# Configure git identity (add to .env)
GIT_AUTHOR_NAME=Your Name
GIT_AUTHOR_EMAIL=your@email.com

# Commit from container (hooks run correctly)
docker compose exec hogball git add -A
docker compose exec hogball git commit -m "Your commit message"

# Push from host (uses your SSH keys)
git push
```

### Supabase Keep-Alive

Supabase Cloud free tier auto-pauses after 7 days. If paused:

```bash
docker compose exec hogball pnpm run prime
```

## Forking This Repository

When forking HogBall (or any ScriptHammer-based project) to create your own project:

### Quick Start

```bash
# 1. Fork and clone
gh repo fork TortoiseWolfe/HogBall --clone
cd YourProject

# 2. CRITICAL: Stop Docker first (service name will change)
docker compose down

# 3. Run rebrand script
./scripts/rebrand.sh YourProject yourusername "Your project description"

# 4. Create .env file
cp .env.example .env
# Edit .env with your UID/GID (run: id -u && id -g)

# 5. Rebuild with new service name
docker compose up --build

# 6. Verify
docker compose exec yourproject pnpm run build
docker compose exec yourproject pnpm test -- --run
```

### Manual Review After Rebrand

The rebrand script handles 95% of changes automatically, but review these manually:

1. **Footer.tsx** - The script changes the template attribution link. Decide whether to:
   - Keep pointing to ScriptHammer (recommended - give credit to the template)
   - Update to your own project

2. **FORKING-FEEDBACK.md** - Add your learnings for future forkers

3. **.env** - Configure your Supabase credentials for full functionality

### Known Quirks

- The rebrand script updates ALL owner/project references, including template attribution
- Docker service name changes require `docker compose down` first
- Contract/integration tests require Supabase credentials (unit tests work without)
- CLAUDE.md docker commands are automatically updated by the script

### Files Modified by Rebrand Script

| Category | Count | Examples |
|----------|-------|----------|
| Config | 10+ | docker-compose.yml, package.json, .env.example |
| Source | 50+ | Components, services, configs |
| Docs | 100+ | All markdown files |
| Tests | 10+ | Fixtures, contract tests |
| Total | ~350 | All ScriptHammer → YourProject |

See `docs/FORKING-FEEDBACK.md` for detailed lessons learned from previous forks.

## Component Structure (MANDATORY)

Components must follow the 5-file pattern or CI/CD will fail:

```
ComponentName/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TortoiseWolfe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
