---
trigger: always_on
description: > Project guide for AI coding agents working on SkillNote.
---

# CLAUDE.md

> Project guide for AI coding agents working on SkillNote.

## Overview

SkillNote is a self-hosted skill registry for AI coding agents. It lets teams create, version, and distribute `SKILL.md` files across Claude Code, Cursor, Codex, OpenHands, and more. The app is offline-first: skills are stored in localStorage and sync to PostgreSQL when the backend is available.

## Tech Stack

| Layer    | Technology                                                    |
| -------- | ------------------------------------------------------------- |
| Frontend | Next.js 16, React 19, TypeScript, Tailwind CSS 4, Tiptap     |
| Backend  | Python 3.12, FastAPI, SQLAlchemy 2, Alembic, Pydantic 2      |
| Database | PostgreSQL 16                                                 |
| CLI      | Node.js, TypeScript, Commander.js                             |
| Infra    | Docker, Docker Compose                                        |

## Project Structure

```
skillnote/
├── src/                          # Next.js frontend (App Router)
│   ├── app/(app)/                # Route group: pages (home, skills, tags, collections, settings)
│   ├── components/
│   │   ├── skills/               # SkillDetail, SkillCard, editor, tabs (view/edit/history/comments)
│   │   ├── layout/               # Sidebar, TopBar, ConnectionBanner
│   │   ├── import/               # ImportModal (SKILL.md file import)
│   │   └── ui/                   # shadcn/ui primitives
│   └── lib/
│       ├── api/client.ts         # Fetch wrapper (apiRequest<T>) + SkillNoteApiError
│       ├── api/skills.ts         # All typed API calls (CRUD, comments, tags, versions)
│       ├── skills-store.ts       # Offline-first state (localStorage + API sync)
│       ├── skill-validation.ts   # Frontend validation (mirrors backend rules exactly)
│       ├── mock-data.ts          # TypeScript types + seed data
│       ├── markdown-utils.ts     # SKILL.md generation + parsing with YAML frontmatter
│       ├── hooks.ts              # useKeyboardShortcut, useClipboard, useLocalStorage
│       └── utils.ts              # cn() (clsx + tailwind-merge)
├── backend/                      # FastAPI backend
│   ├── app/api/                  # Route handlers (skills, comments, tags, publish, downloads)
│   ├── app/db/models/            # SQLAlchemy models (Skill, SkillVersion, SkillContentVersion, Comment)
│   ├── app/schemas/              # Pydantic request/response schemas
│   ├── app/validators/           # skill_validator.py (name/desc rules), bundle_validator.py (ZIP validation)
│   ├── app/core/config.py        # pydantic-settings with SKILLNOTE_ env prefix
│   ├── app/services/             # LocalBundleStorage (ZIP file storage)
│   ├── alembic/versions/         # 4 migrations (initial → rich fields → content versions → drop auth)
│   ├── tests/                    # pytest unit + integration tests
│   └── scripts/                  # seed_data.py, wait_for_db.py, smoke_test.sh
├── cli/                          # CLI tool (skillnote binary)
│   └── src/
│       ├── commands/             # login, list, add, check, update, remove, doctor
│       └── agents/               # Agent adapters (claude, cursor, codex, openclaw, openhands, universal)
├── e2e/                          # Playwright E2E tests
├── docker-compose.yml            # Full stack: postgres + api + web
└── Dockerfile                    # Next.js multi-stage build
```

## Git Workflow

The `master` branch is protected. All changes must go through a feature branch and a pull request. Never push directly to `master`.

1. Create a feature branch (`git checkout -b feat/my-feature`)
2. Commit changes to the feature branch
3. Push the branch and create a PR (`gh pr create`)
4. Merge after approval

## Development Setup

### Full stack with Docker

```bash
docker compose up --build -d
# Web: http://localhost:3000  |  API: http://localhost:8082
```

### Frontend hot-reload

```bash
docker compose up --build -d postgres api    # Backend in Docker
npm install && npm run dev                   # Frontend on localhost:3000
```

### LAN access

```bash
SKILLNOTE_HOST=<your-server-ip> docker compose up --build -d
```

## Key Architectural Patterns

### State Management (No Library)

Skills state lives in `localStorage` under `skillnote:skills`. No Redux, Zustand, or Context.

- `getSkills()` / `writeStorage()` read/write localStorage directly
- Cross-component sync via `window.dispatchEvent(new Event('skillnote:skills-changed'))`
- Home page re-syncs on: mount, 30s interval, window focus, `skillnote:skills-changed` event
- Connection status is a module-level pub/sub (`_connectionStatus` + `_listeners`)

### Offline-First Pattern

1. Render immediately from localStorage (no loading flash)
2. `syncSkillsFromApi()` merges API + local-only skills in the background
3. Create/update: try API first, fall back to local with version increment
4. Delete: API required (no offline delete)
5. `ConnectionBanner` shows when offline with retry button

### API Call Chain

```
Components → skills-store.ts → api/skills.ts → api/client.ts (apiRequest<T>)
```

Components never call API directly (except comments and tags).

### API URL Resolution


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luna-prompts/skillnote](https://github.com/luna-prompts/skillnote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
