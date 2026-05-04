---
trigger: always_on
description: **CRITICAL**: When delegating work to agents, ALWAYS use TeamCreate + Task with `team_name` to spawn **teammates**. NEVER use the Task tool as a standalone single agent. Every piece of work must go through a team:
---

# Points: GenLayer Testnet Program Tracking System

## ALWAYS Use Teammates (Teams), NEVER Single Tasks
**CRITICAL**: When delegating work to agents, ALWAYS use TeamCreate + Task with `team_name` to spawn **teammates**. NEVER use the Task tool as a standalone single agent. Every piece of work must go through a team:
1. Create a team with `TeamCreate`
2. Spawn teammates with `Task` using `team_name` parameter
3. Coordinate via `SendMessage` and `TaskList`
4. Shut down teammates and `TeamDelete` when done

This applies to ALL work — backend, frontend, research, everything. No exceptions.

## 📚 Quick Reference Documentation
**Important**: This project has detailed documentation for faster development:
- **Backend Documentation**: See `backend/CLAUDE.md` for Django structure, API endpoints, models, and patterns
- **Frontend Documentation**: See `frontend/CLAUDE.md` for Svelte 5 structure, routes, components, and API integration
- **Session Management**: See `SESSIONS.md` for the Git worktree-based development session system

⚠️ **Keep documentation updated**: When making changes to the codebase, update the relevant CLAUDE.md file to help future development sessions.

## Development Sessions & Worktrees
This project uses a custom session management system with Git worktrees. See `SESSIONS.md` for complete documentation on:
- Creating and managing parallel development sessions
- Port allocation and tmux window management
- Claude Code session persistence across worktrees
- Troubleshooting common issues

Quick commands:
```bash
./session-mgmt add <name>     # Create new session
./session-mgmt list            # Show all sessions
./session-mgmt resume [name]   # Resume after restart
./session-mgmt remove <name>   # Remove session
```

## Svelte 5 Important Notes

### Runes Mode and Props
The frontend uses Svelte 5 with runes mode enabled. This means:
- **DO NOT use `export let` for props** - This will cause an error
- **ALWAYS use `$props()` instead** for component props

```javascript
// ❌ WRONG - This will cause an error in runes mode
export let params = {};

// ✅ CORRECT - Use $props() in Svelte 5
let { params = {} } = $props();
```

## Git Workflow & Guidelines

### Branch Strategy
**Important**: The `dev` branch is the main merging branch for all development work.
- Always create pull requests targeting `dev`, not `main`
- The `main` branch is reserved for production releases
- Feature branches should be based on and merged into `dev`

### Commits and PRs
**Before any commit or PR**, read and follow `.claude/commands/commit.md`. It defines the commit message format, changelog workflow, and git history rules. No exceptions.

## Important Terminology

### Never Use "Badge" - Always Use "Contribution"
**IMPORTANT**: The word "badge" should NEVER be used in the UI or code comments. Always use "contribution" instead:
- ❌ WRONG: "Claim your badge", "Badge earned", "Badge requirements"
- ✅ CORRECT: "Complete your journey", "Contribution earned", "Journey requirements"
- ❌ WRONG: "Builder Badge", "Validator Badge"
- ✅ CORRECT: "Builder Welcome Contribution", "Validator Waitlist Contribution"

The system tracks contributions, not badges. Each journey completion results in a contribution being recorded.

## Environment Setup

### Frontend Development
When working with the frontend, always activate the Node.js environment before using npm commands:

```bash
# Activate the environment before using npm
cd [..]/points/frontend
source ../backend/env/bin/activate
# Then run npm commands
npm install
npm run dev
npm test
```

### Backend Development
For backend development, activate the Python virtual environment:

```bash
# Activate the Python virtual environment
cd [..]/points/backend
source env/bin/activate
# Then run Python/Django commands
python manage.py runserver
```

## Project Overview

Points is a points and contribution tracking system specifically designed for the 
GenLayer Foundation's Testnet Program. It serves as a comprehensive platform that 
tracks, rewards, and visualizes community contributions to the GenLayer ecosystem. 
Each participant action earns badges and points, with the system applying dynamic 
multipliers to transform action points into global points that determine rankings 
on the leaderboard.

The system represents an experiment in creating autonomous and trustless GLF 
(GenLayer Foundation) programs that will eventually become part of the 
Deepthought DAO.

## Technical Architecture

### Backend (Python/Django)

The backend is built with Django and Django REST Framework, providing a robust 
API for tracking and managing contributions. The architecture consists of several 
key components:

1. **Main Django Apps**:
   - **users**: Custom user management system with email-based authentication
   - **contributions**: Tracking different types of contributions and their 
     associated points
   - **leaderboard**: Global ranking system with dynamic multipliers
   - **utils**: Shared utilities and base models
   - **api**: Core API functionality

2. **Database Models**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [genlayer-foundation/points](https://github.com/genlayer-foundation/points) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
