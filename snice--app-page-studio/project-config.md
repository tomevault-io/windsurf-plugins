---
trigger: always_on
description: @/Users/itfenbao/.codex/RTK.md
---

@/Users/itfenbao/.codex/RTK.md

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

App Page Studio is a web tool for turning design inputs into structured AI implementation prompts for Flutter, React Native, and UniApp. It supports HTML exports, raster design images, PSD previews/layers/slices, AI-generated HTML IR previews and refinements, page grouping, design-system metadata, and multi-user project collaboration.

The active frontend is Vite + React under `packages/client/`. The backend is under `packages/server/`. The old root-level `public/` HTML/CSS/JS implementation is obsolete and should not be updated.

## Commands

Agent shell commands in this repo should be run with the `rtk` prefix.

```bash
rtk pnpm run dev    # Install dependencies, then start Express and Vite concurrently
rtk pnpm run build  # Install dependencies, build client, and create release ZIP
```

Useful maintenance command:

```bash
rtk pnpm --filter server reset-password -- -u <username>
```

AI HTML IR smoke test:

```bash
rtk pnpm --filter server test-ai-html-agent -- --projectId 1 --path __design__/figma_page_d8e2c82aab.png
```

## Architecture

### Backend Structure

```text
packages/server/
├── server.js           # Express entry, sessions, static frontend, WebSocket, file watcher
├── db.js               # SQLite schema and data-access modules
├── paths.js            # Workspace/data/build output paths
└── api/
    ├── auth.js         # Login/logout/current user/admin user management
    ├── projects.js     # Project CRUD and project members
    ├── pages.js        # Pages config save/load APIs
    ├── html.js         # HTML/design file upload, scan, delete, ZIP download
    ├── image.js        # Design image and asset upload/list APIs
    ├── psd.js          # PSD upload/list/preview APIs
    ├── ai-html-agent.js # PNG/PSD preview -> HTML IR generation/refinement
    ├── prompt.js       # Prompt generation route
    ├── prompt/         # Prompt builders by target platform
    └── utils.js        # Shared upload, auth guard, path, ZIP, broadcast helpers
```

### Server (`server.js`)

- Uses Express with JSON payloads up to 50 MB.
- Uses `express-session` with `better-sqlite3-session-store`; cookie name is `aps.sid`.
- Bootstraps an admin account on first run. `BOOTSTRAP_ADMIN_USERNAME` and `BOOTSTRAP_ADMIN_PASSWORD` can override defaults.
- Serves Vite build output from `frontend_dist` first, then `packages/client/dist`.
- Serves project files from `/html/:projectId` after auth and project-access checks.
- Mounts `/api/auth/*` first; all business API routers are protected by `requireAuth`.
- Provides SPA fallback to `index.html` for non-API and non-HTML routes.
- Runs authenticated WebSocket upgrades at `/ws`.
- Watches `html_caches/` with chokidar and broadcasts `html:changed` for HTML/PSD changes.

### WebSocket Model

WebSocket connections are authenticated through the same Express session as HTTP.

Current event types:

- `session` - server returns session, connection, and user identity.
- `presence:update` - client reports current project/page/group scope.
- `presence:list` - server broadcasts current collaborators in a project.
- `files:changed` - upload/delete changed file list; other clients rescan.
- `pages:file-saved` - one page config was saved; other clients merge when clean.
- `pages:groups-saved` - group metadata and file assignments were saved.
- `pages:full-saved` - whole project config was saved.
- `html:changed` - watched HTML/PSD file changed on disk.

Presence is advisory: it helps users see possible page/group conflicts before saving. Actual conflict control is enforced by revision/hash checks in `api/pages.js`.

### Database (`db.js`)

SQLite is accessed through `better-sqlite3`.

Core tables:

- `users`: username, password hash, role (`admin` or `user`).
- `projects`: name, description, design system JSON, owner user.
- `project_members`: project/user membership with role (`owner`, `editor`, `viewer`).
- `project_pages`: current pages config JSON, revision, updated actor/session.

The session store also persists Express session data in SQLite.

Key data modules:

- `Users`: login/admin user management helpers.
- `Projects`: project CRUD, access checks, members, and page config save/merge helpers.

### API Modules

All business APIs require login unless noted.

**auth.js**

- `POST /api/auth/login`
- `POST /api/auth/logout`
- `GET /api/auth/me`
- `GET /api/auth/users` - admin only
- `POST /api/auth/users` - admin only
- `PUT /api/auth/users/:id` - admin only
- `DELETE /api/auth/users/:id` - admin only

**projects.js**

- `GET /api/projects` and `GET /api/config` - list accessible projects.
- `GET /api/projects/:id` - get one project.
- `GET /api/projects/:id/members` - list members and, for managers, users.
- `POST /api/projects/:id/members` - add/update member.
- `PUT /api/projects/:id/members/:userId` - update member role.
- `DELETE /api/projects/:id/members/:userId` - remove member.
- `POST /api/projects` - create project, optionally with ZIP upload.
- `PUT /api/projects/:id` - update name, description, design system.
- `POST /api/projects/:id/html` - replace the project's `__html__` directory.
- `DELETE /api/projects/:id` - delete project.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [snice/app-page-studio](https://github.com/snice/app-page-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-20 -->
