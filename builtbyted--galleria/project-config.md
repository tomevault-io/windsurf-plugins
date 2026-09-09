---
trigger: always_on
description: Photography portfolio website with React 19, TypeScript, Express 5, and SQLite.
---


# Galleria - Project Rules

## Project Overview

Photography portfolio website with React 19, TypeScript, Express 5, and SQLite.

**Architecture**: Monorepo with separate frontend and backend. SQLite for data, filesystem for photos.

**Deployment**: PM2 for production, separate dev servers for development.

## Development

### Server Management

**Development**: DO NOT use PM2 for development. Run backend and frontend separately:

- Backend: `cd backend && npm run dev`
- Frontend: `cd frontend && npm run dev`

**Production**: PM2 is used for production only. Use `./restart.sh` or `pm2 start ecosystem.config.js`

### Build Process

**Backend**: Backend changes require `npm run build` to compile TypeScript before deployment.

**Frontend**: Frontend uses Vite for fast HMR in development.

**Favicons**: During deployment, `scripts/generate-favicons.js` checks for custom avatar in `data/photos/avatar.png` and generates all icons and favicons from it. If no custom avatar exists, falls back to defaults from `config/icons/`. This ensures custom favicons persist across deployments since `data/` is not gitignored while `frontend/public/*.png` files are.

### Testing

**Browser Tools**: NEVER use browser tools (browser_navigate, browser_snapshot, browser_click, etc.). NEVER open Chrome or any browser window for testing.

**Rationale**: The user prefers to test manually in their own browser. Opening browser windows is disruptive to their workflow.

## Critical Patterns

### Database First

**Rule**: ALWAYS query SQLite database instead of scanning filesystem.

**Rationale**: Filesystem scans are slow and don't scale. Use `image_metadata` table for images, `albums` table for albums.

**Examples**:

- Use `getImagesInAlbum(album)` instead of `fs.readdirSync()`
- Use `getAllAlbums()` instead of scanning photos directory
- Use `getImagesFromPublishedAlbums()` for published content

**Exceptions**: Only scan filesystem when syncing new uploads to database.

### SSE Race Conditions

**Rule**: For SSE (Server-Sent Events) endpoints, create job tracking IMMEDIATELY after checking if job exists.

**Rationale**: Prevents race condition where two simultaneous requests both think no job exists and spawn duplicate processes.

**Pattern**: Check if job exists → Create job object → Set headers → Spawn process

**Anti-pattern**: Never set headers or start any work before creating the job tracking object.

**Files**: `backend/src/routes/ai-titles.ts`, `backend/src/routes/image-optimization.ts`

### SSE Broadcasting

**Rule**: SSE handlers must broadcast to ALL connected clients and store output history.

**Rationale**: Multiple clients can connect/reconnect. All must receive updates and see previous output.

**Pattern**:

- Store output in `job.output` array
- Use `broadcastToClients(job, message)` for all output (stdout, stderr, errors)
- Never write directly to a single `res` object in data handlers

**Files**: `backend/src/routes/ai-titles.ts`, `backend/src/routes/image-optimization.ts`

### SSE Timeouts

**Rule**: SSE connections must disable response timeouts and proxy buffering.

**Rationale**: Prevents network timeout errors on long-running operations (uploads, optimization, AI generation).

**Pattern**:

- `res.setHeader('X-Accel-Buffering', 'no')` to disable proxy buffering
- `res.setTimeout(0)` to disable response timeout
- Apply to ALL SSE endpoints immediately after setting other headers

**Files**: `backend/src/routes/ai-titles.ts`, `backend/src/routes/image-optimization.ts`, `backend/src/routes/album-management.ts`

### Authentication

**Rule**: Use `requireAuth` middleware for all admin endpoints.

**Implementation**: `req.isAuthenticated()` checks if user is logged in via Google OAuth.

**Files**: All routes in `backend/src/routes/` except public endpoints (albums, photos, sitemap, health).

### Role-Based Access Control (RBAC)

**Rule**: Enforce role-based permissions across the application.

**Roles**:

- `viewer` - Read-only access to content
- `manager` - Full content editing, no system settings
- `admin` - Full access to everything

**Backend Middleware**:

- `requireAuth` - User must be authenticated (any role)
- `requireManager` - User must be manager or admin
- `requireAdmin` - User must be admin

**Viewer Permissions**:

- ✅ View all albums and photos (including unpublished)
- ✅ View metrics page
- ✅ Access profile page (manage own account: password, MFA, passkeys)
- ❌ Cannot edit albums, photos, or content
- ❌ Cannot access settings page
- ❌ Cannot see other users

**Manager Permissions**:

- ✅ Full edit access to albums and photos
- ✅ Can create, rename, delete albums and folders
- ✅ Can upload, edit, delete photos
- ✅ Can manage branding and external links
- ✅ Can run AI title generation and image optimization
- ✅ View all albums and photos (including unpublished)
- ✅ View metrics page
- ✅ Access profile page (manage own account only)
- ❌ Cannot access system settings (config, SMTP, OpenAI, user management)
- ❌ Cannot see other users

**Admin Permissions**:

- ✅ Full access to everything
- ✅ Access settings page (config, SMTP, OpenAI, user management)
- ✅ Can manage all users (invite, delete, reset MFA, change roles)
- ✅ Can modify system configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BuiltByTed/Galleria](https://github.com/BuiltByTed/Galleria) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
