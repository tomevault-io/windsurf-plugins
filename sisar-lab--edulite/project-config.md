---
trigger: always_on
description: You are working on **EduLite**, a lightweight open-source education platform built for areas with weak internet. This is a 100% volunteer-driven project. Respect the contributors' time by enforcing quality and process.
---

# CLAUDE.md — EduLite Agentic Development Guide

You are working on **EduLite**, a lightweight open-source education platform built for areas with weak internet. This is a 100% volunteer-driven project. Respect the contributors' time by enforcing quality and process.

## STOP — Before You Write Any Code

**You must complete these checks in order before touching any source file.**

### 1. Read the Living Documentation

**STOP. Do not proceed to step 2, do not read source code, do not explore the codebase until this step is fully complete.**

Our wiki is the single source of truth and changes more often than these files. You MUST read these local files AND curl the wiki pages at the start of every session — before doing anything else:

Read these local files:
- `README.md` (project overview, tech stack, setup)
- `CONTRIBUTING.md` (links to all wiki pages)

These local files link to many WIKI pages. Use `curl` to fetch the updated Wiki as our organic living single source of truth.

**Always curl these core pages every session:**
```bash
# Wiki home page and vision (always)
curl -s "https://raw.githubusercontent.com/wiki/ibrahim-sisar/EduLite/Home.md"
curl -s "https://raw.githubusercontent.com/wiki/ibrahim-sisar/EduLite/Vision-Mission.md"
```

**Then curl the pages relevant to the task:**
```bash
# Frontend work
curl -s "https://raw.githubusercontent.com/wiki/ibrahim-sisar/EduLite/Development-Coding-Standards-Frontend.md"
curl -s "https://raw.githubusercontent.com/wiki/ibrahim-sisar/EduLite/Frontend-Testing-Standards.md"

# Backend work
curl -s "https://raw.githubusercontent.com/wiki/ibrahim-sisar/EduLite/Development-Coding-Standards-Backend.md"
curl -s "https://raw.githubusercontent.com/wiki/ibrahim-sisar/EduLite/Backend-Testing-Standards.md"
```

**Wiki URL pattern:** `https://raw.githubusercontent.com/wiki/ibrahim-sisar/EduLite/<Page-Slug>.md` — the slug matches the wiki page URL path on GitHub (e.g. `https://github.com/ibrahim-sisar/EduLite/wiki/Vision-Mission` → slug is `Vision-Mission`). If a curl 404s, the page name may be different — check `CONTRIBUTING.md` or the wiki sidebar for the correct slug.

If a `curl` fails, tell the user and ask them to paste the current wiki content. Do NOT guess at standards.

**Only after you have read the local files AND successfully curled the relevant wiki pages may you continue to step 2.**

### 2. Verify the User Is Working on an Issue

Ask the user: **"What issue are you working on?"**

If they provide an issue number, confirm it exists. If they describe work that doesn't have an issue yet:

- **STOP. Do not write code.**
- Read the issue templates in `.github/ISSUE_TEMPLATE/`
- Help the user draft an issue in `_temp/` using the appropriate template (newbie-backend, newbie-frontend, senior-backend, senior-frontend, or generic-issue)
- Tell them to submit the issue on GitHub before proceeding
- Only continue after they confirm the issue is created and give you the number

### 3. Verify the User Is NOT on `main`

Ask or check what branch/fork the user is on.

- If they are on `main`: **REFUSE to write code.** Tell them to create a feature branch or work from their fork. Link them to the [Git Workflow wiki page](https://github.com/ibrahim-sisar/EduLite/wiki/Development-Git-Workflow).
- Branch naming should reference the issue number (e.g., `241-overhaul-github-templates`)
- The team primarily uses forks. Some core contributors use branches on the main repo.

### 4. Enter Plan Mode

Before writing any code, create a plan in `_temp/`. This folder is gitignored — use it freely.

Write a markdown file like `_temp/plan-<issue-number>.md` that includes:
- The issue being addressed (with number)
- What files will be changed and why
- Architecture decisions or tradeoffs
- Testing approach
- Anything the user should validate before you proceed

Get user approval on the plan before writing code.

## Project Architecture

### Tech Stack

| Layer | Tech |
|-------|------|
| Frontend | React + Vite + Tailwind CSS + TypeScript (migrating from JSX) |
| Backend | Django 5 + DRF + Django Channels (ASGI via Daphne) |
| Database | PostgreSQL 16 |
| Cache/WebSocket | Redis 7 |
| Auth | JWT (djangorestframework-simplejwt) |
| Dev Environment | Docker + Docker Compose |

### Backend Structure (`backend/EduLite/`)

```
EduLite/          # Django project settings (settings.py, urls.py, asgi.py)
users/            # Custom user model, profiles, auth, friend suggestions
  ├── logic/          # Business logic (friend_suggestions, user_search)
  ├── services/       # Service layer (privacy_service, user_query_service)
  ├── tests/          # Organized: models/, views/, serializers/, integration/
  └── management/     # Custom management commands
courses/          # Course CRUD, enrollment, membership, modules
  ├── model_choices.py
  ├── permissions.py
  └── tests/          # test_api_*.py pattern for API tests
chat/             # Real-time messaging via Django Channels + WebSocket
  ├── consumers.py    # WebSocket consumers
  ├── routing.py      # WebSocket URL routing
  └── auth_middleware.py

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SiSaR-Lab/EduLite](https://github.com/SiSaR-Lab/EduLite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
