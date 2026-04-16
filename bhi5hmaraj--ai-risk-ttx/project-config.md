---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Simulacra** is an AI-powered tabletop exercise (TTX) simulation game built with React, TypeScript, and Vite. Named after Jean Baudrillard's concept of simulations that become more "real" than reality itself, players assume strategic roles during an AI-driven global crisis, making decisions that affect public trust and secret objectives. The game is driven by LLM API calls (via LiteLLM proxy) that act as a Game Master, generating dynamic scenarios, consequences, and AI opponent actions.

## Development Commands

### Setup
```bash
npm ci                    # Install dependencies (recommended)
npm install              # Install dependencies

# Database Setup (one-time, local development)
npm run db:setup         # Automated PostgreSQL setup + migrations

# Or manual Prisma setup
npx prisma generate      # Generate Prisma client
npx prisma migrate dev   # Run database migrations (requires DATABASE_URL)
```

### Development
```bash
npm run dev              # Start Vite dev server (frontend only)
./dev.sh                 # Start with Vercel (frontend + API routes)
npm run build            # Production build
npm run preview          # Preview production build locally

# Database management
npm run db:studio        # Open Prisma Studio (database GUI)
npm run db:migrate       # Create new migration
npm run db:reset         # Reset database (⚠️ deletes all data)

# Analytics
npm run analyze          # Analyze feedback data
npm run analyze -- --help # Show all analytics options

# Testing
npm run test:api         # Test feedback API endpoint
```

### Issue Tracking with Beads

This project uses **Beads** (bd) for dependency-aware issue tracking. Issues are chained together like beads, with explicit dependencies to prevent duplicate work and ensure proper sequencing.

**Key Commands**:
```bash
bd quickstart              # Learn about Beads
bd list                    # List all issues
bd list --status open      # List open issues
bd ready                   # Show issues ready to work on (no blockers)
bd show ai-risk-ttx-15     # Show issue details
bd create "Task name"      # Create new issue
bd dep add bd-1 bd-2       # Add dependency (bd-2 blocks bd-1)
bd dep tree bd-1           # Visualize dependency tree
bd update bd-1 --status in_progress  # Update issue status
bd close bd-1              # Close completed issue
```

**Database Location**: `.beads/issues.db` (auto-syncs with `.beads/issues.jsonl` for git)

**Integration with README**:
The README.md auto-updates with Beads statistics via pre-commit hook (`scripts/update_readme.py`):
- Issue counts by status and priority
- Ready-to-work issues (no blockers)
- Mermaid dependency graph

**For Claude Code**:
- Use `bd ready` to find unblocked work
- Create issues when discovering new tasks: `bd create "Task description" -p 1`
- Add dependencies to prevent out-of-order work: `bd dep add child parent`
- Update status when starting work: `bd update <id> --status in_progress`
- Close issues when complete: `bd close <id>`

### Pre-Commit Workflow
**IMPORTANT**: Before committing, use the helper script to ensure clean lockfiles and passing builds:

```bash
chmod +x ./git-push.sh
./git-push.sh                           # Prepare repo (regen lockfile, validate, build)
./git-push.sh -c "Your commit message"  # Auto-commit and push
```

This script:
- Validates Node >= 20
- Regenerates `package-lock.json` deterministically
- Validates install with `npm ci`
- Builds with safe env defaults
- Stages updated lockfile
- Pre-commit hook runs `scripts/update_readme.py` to sync Beads stats

## Environment Configuration

Required environment variables (set in `.env` or Vercel):

```bash
# LLM Configuration
VITE_LITELLM_API_KEY    # LiteLLM proxy API key (virtual/master key)
VITE_LLM_MODEL          # Model name (e.g., "gemini-2.5-flash", "gpt-4o-mini")

# Database (for feedback and public scenarios)
DATABASE_URL            # PostgreSQL connection string
                        # Local: "postgresql://user:pass@localhost:5432/dbname"
                        # Vercel: Automatically provided by Vercel Postgres
```

The LiteLLM base URL is hardcoded in `services/geminiService.ts:25` as `https://asgard.bhishmaraj.org`.

See `prisma/README.md` for detailed database setup instructions.

## Architecture

### Core Technologies
- **React 19** with hooks-based state management
- **TypeScript** (strict mode via tsconfig.json)
- **Vite** for build tooling
- **OpenAI SDK** for LLM calls (via LiteLLM proxy)
- **Zod** for schema validation and structured outputs
- **React Flow** for action tree visualization
- **Prisma** with PostgreSQL for data persistence (feedback, public scenarios)

### Project Structure

```
/
├── App.tsx                 # Main app component, phase-based routing
├── index.tsx              # React root
├── types.ts               # Core TypeScript types and enums
├── constants.tsx          # Game config and role definitions
├── prompts.ts             # LLM prompt templates and schemas
├── presets.ts             # Pre-built scenarios (e.g., AI_SAFETY_SCENARIO)
├── hooks/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bhi5hmaraj) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
