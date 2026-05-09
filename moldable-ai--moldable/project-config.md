---
trigger: always_on
description: Moldable is a personal software factory - a desktop application where users prompt an AI agent to create, shape, and discard hyper-personalized applications directly on their local machine.
---

# Moldable Development Guidelines

## Project Overview

Moldable is a personal software factory - a desktop application where users prompt an AI agent to create, shape, and discard hyper-personalized applications directly on their local machine.

**Core concept**: Software is summoned, shaped, and discarded through natural language conversation.

**Key mechanics**:

- Users interact via a chat interface that hovers over all views
- The AI agent creates apps that run as local servers
- Apps are displayed within the desktop app via webviews
- Each app has a **widget view** (glanceable) and **full view** (expanded)
- Skills, MCPs, and sub-agents extend capabilities over time

See `prds/moldable.prd.md` for the complete product specification.

## Directory Structure

Moldable uses a **workspace-based** structure where data is isolated per workspace:

```
~/.moldable/                         # User data directory (MOLDABLE_HOME)
├── workspaces.json                  # Workspace list + active workspace
│
├── shared/                          # Shared across ALL workspaces
│   ├── .env                         # Base environment variables (API keys)
│   ├── apps/                        # App source code (shared across workspaces)
│   │   ├── scribo/                  # Installed app source
│   │   │   ├── moldable.json
│   │   │   ├── package.json
│   │   │   └── src/
│   │   └── meetings/
│   ├── scripts/                     # Shared scripts (lint-moldable-app.js, etc.)
│   ├── skills/                      # Skills library (instruction & executable)
│   │   └── {repo-name}/             # Skills grouped by source repo
│   │       └── {skill-name}/        # Individual skill (SKILL.md or bin/)
│   ├── mcps/                        # Custom MCP server code
│   │   └── {mcp-name}/              # e.g., "my-api-gateway"
│   │       ├── server.js            # MCP server (stdio)
│   │       └── package.json
│   └── config/
│       └── mcp.json                 # Shared MCP server connections
│
└── workspaces/                      # Per-workspace isolated data
    └── {workspace-id}/              # e.g., "personal", "work"
        ├── config.json              # Apps enabled, preferences
        ├── .env                     # Workspace-specific env overrides
        ├── apps/                    # App runtime data (workspace-scoped)
        │   ├── {app-id}/            # E.g. data for this app in this workspace
        │   │   └── data/            # App runtime data (SQLite, files, etc.)
        ├── conversations/           # Chat history
        └── config/
            ├── mcp.json             # Workspace-specific MCPs
            └── skills.json          # Which shared skills are enabled

/Users/{user}/moldable/              # Development workspace (monorepo)
├── desktop/                         # Tauri desktop app
├── packages/                        # Shared npm packages (@moldable-ai/*)
└── prds/                            # Product specifications

/Users/{user}/moldable-apps/         # Official apps repository (separate repo)
├── manifest.json                    # App registry for discovery
├── scribo/                          # Translation journal app
├── meetings/                        # Meeting recorder app
└── ...                              # Other official apps
```

### Workspaces

Workspaces allow isolating data between contexts (Personal, Work, Side Project, etc.):

- **Instant switching**: All apps from all workspaces run simultaneously
- **Shared apps**: App code is installed once in `shared/apps/`, data is per-workspace
- **Shared skills**: Skills are installed once, enabled per-workspace
- **Layered .env**: `shared/.env` provides base values, workspace `.env` overrides

## Tech Stack

### Desktop App

- **Framework**: Tauri v2 (Rust backend + Web frontend)
- **Frontend**: Vite + React 19 + TypeScript
- **Styling**: Tailwind CSS 4 + shadcn/ui
- **Package Manager**: pnpm

### Generated Apps (Default)

- **Framework**: Vite + Hono + React 19 + TypeScript
- **Styling**: Tailwind CSS 4 + shadcn/ui
- **Database**: SQLite (local) or Postgres (via Tilt/Docker)
- **API Layer**: Hono same-origin `/api/*` routes
- **Package Manager**: pnpm

Note: The agent may use different stacks based on requirements (e.g., Python for data analysis).

## Development Commands

### Desktop

```bash
cd desktop
pnpm dev              # Run Vite dev server
pnpm tauri dev        # Run full Tauri app
pnpm tauri build      # Build for production
```

### Code Quality Checks

**IMPORTANT**: After completing code changes, ALWAYS run these from the repo root:

```bash
pnpm lint             # Run ESLint on all packages
pnpm check-types      # Run TypeScript compiler to check for errors
pnpm test             # Run tests (if applicable to the changed package)
```

Run these proactively to catch issues before the user does. Don't wait for errors to be reported.

## General Rules

- **kebab-case** for all file and directory names
- Split files into smaller parts - don't dump everything into single files
- Always install packages with `pnpm add` (or `pnpm add -D` for dev)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moldable-ai/moldable](https://github.com/moldable-ai/moldable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
