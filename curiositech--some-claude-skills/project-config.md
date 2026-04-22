---
trigger: always_on
description: A curated gallery of 90+ Claude Code skills with a distinctive Windows 3.1 retro aesthetic. Built on Docusaurus, deployed to GitHub Pages.
---

# CLAUDE.md - Some Claude Skills

## Project Overview

A curated gallery of 90+ Claude Code skills with a distinctive Windows 3.1 retro aesthetic. Built on Docusaurus, deployed to GitHub Pages.

**Live Site**: https://someclaudeskills.com
**Repository**: https://github.com/erichowens/some_claude_skills

---

## Active Development: Platform Enhancement

We are transforming this from a documentation site into **the premier curated Claude Code skills platform** with:
- Tutorial-first learning
- Skill bundles for common workflows
- Video walkthroughs
- Improved onboarding

### Project Tracking

All project tracking lives in `.project/`:

```
.project/
├── ROADMAP.md           # Current sprint + future work
├── PROGRESS.md          # Daily progress log
├── BUGS.md              # Known issues and blockers
├── CHANGELOG.md         # Completed work (dated)
├── DECISIONS.md         # Architecture decisions (ADRs)
└── PARALLEL.md          # Worktree parallelization guide
```

### Key Documents

| Document | Location | Purpose |
|----------|----------|---------|
| Master Plan | `docs/MASTER_IMPLEMENTATION_PLAN.md` | Strategic synthesis from 6 agents |
| Task List | `docs/SEQUENTIAL_TASK_LIST.md` | Detailed implementation tasks |
| UX Analysis | `docs/UX_DEEP_ANALYSIS.md` | Gestalt + Markov chain analysis |
| Go-to-Market | `docs/GO_TO_MARKET_STRATEGY.md` | Launch strategy |
| User Personas | `docs/USER_PERSONAS.md` | 10 target audience profiles for design decisions |

---

## Commands

```bash
cd website/

# Development
npm run start              # Dev server (localhost:3000)
npm run build              # Production build
npm run serve              # Serve production build locally

# Code Quality
npm run lint               # ESLint
npm run typecheck          # TypeScript check
npm test                   # Run Vitest tests (when configured)

# Generation
npm run generate:bundles   # Generate bundles.ts from YAML
npm run generate:skills    # Regenerate skills.ts from markdown
npm run generate:og        # Generate OG images
npm run generate:zips      # Generate skill zip files

# Deployment
npm run deploy             # Deploy to GitHub Pages

# Skills Syncing
npm run sync:user-skills   # Sync project skills to user-level (manual)
# Auto-runs on: git checkout, git pull, git merge
```

---

## Skill Syncing System

**Architecture**: Project skills are the **source of truth** (version controlled), symlinked to user-level for global availability.

### How It Works

1. **Project Skills** (`/coding/some_claude_skills/.claude/skills/`)
   - Version controlled in git
   - Source of truth for all skills
   - 143 skills maintained here

2. **User Skills** (`~/.claude/skills/`)
   - Symlinks to project skills
   - Available globally in all Claude Code sessions
   - Auto-updated via git hooks

### Automatic Sync

Runs automatically on:
- `git checkout` (switching branches)
- `git pull` (pulling changes)
- `git merge` (merging branches)

### Manual Sync

```bash
# From anywhere
npm run sync:user-skills

# Or directly
./scripts/sync-skills.sh
```

### What It Does

1. **Migrates** any skills from user-level to project (one-time)
2. **Removes** stale symlinks from user-level
3. **Creates** fresh symlinks for all project skills

### Verifying

```bash
# Check skill counts
ls -1 .claude/skills/ | wc -l        # Project skills
ls -1 ~/.claude/skills/ | wc -l      # User symlinks (should match)

# Check a specific skill is symlinked
ls -la ~/.claude/skills/computer-vision-pipeline
```

**Benefits**:
- ✅ Skills version controlled with project
- ✅ Available globally across all projects
- ✅ One source of truth
- ✅ No manual copying needed
- ✅ Auto-sync on git operations

---

## Architecture

### Directory Structure

```
some_claude_skills/
├── .project/              # Project tracking (NEW)
├── docs/                  # Strategy & planning documents
├── website/               # Docusaurus site
│   ├── src/
│   │   ├── components/    # React components
│   │   │   ├── win31/     # Win31 design system (NEW)
│   │   │   ├── tutorial/  # Tutorial components (NEW)
│   │   │   ├── bundle/    # Bundle components (NEW)
│   │   │   └── video/     # Video components (NEW)
│   │   ├── hooks/         # Custom React hooks
│   │   ├── types/         # TypeScript interfaces
│   │   ├── data/          # Generated data files
│   │   ├── pages/         # Custom pages
│   │   ├── css/           # Global styles
│   │   └── utils/         # Utility functions
│   ├── docs/              # MDX documentation (skills)
│   ├── bundles/           # Bundle YAML definitions (NEW)
│   └── scripts/           # Build-time scripts
└── skills/                # Raw skill markdown files
```

### Tech Stack

- **Framework**: Docusaurus 3.x
- **UI**: React 18 + TypeScript
- **Styling**: CSS Modules + Win31 design system
- **Analytics**: Plausible (privacy-focused)
- **Hosting**: GitHub Pages
- **Testing**: Vitest + React Testing Library (adding)

---

## Current Sprint

**Phase 1: Foundation (Week 1-2)**

See `.project/ROADMAP.md` for current tasks.

Priority order:
1. Onboarding modal for first-time visitors
2. Tutorial system with progress tracking
3. Skill bundles with one-click install

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [curiositech/some_claude_skills](https://github.com/curiositech/some_claude_skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
