---
trigger: always_on
description: > Essential instructions for AI coding agents working on this project.
---

# AI Agent Development Guide

> Essential instructions for AI coding agents working on this project.
> For initial setup, see `docs/AGENTS_SETUP.md`.

## Project Overview

The project is based on a SaaS starter template built on Cloudflare's edge platform with:
- **Frontend**: React 19 + TypeScript + Vite + Tailwind CSS + shadcn/ui
- **Backend**: Hono on Cloudflare Workers
- **Database**: Cloudflare D1 (SQLite at the edge)
- **Auth**: Better Auth (email/password with verification)
- **Email**: Cloudflare Email Service or Resend (pluggable provider)

## Critical Architecture

### Vite + Hono Integration
The `@cloudflare/vite-plugin` unifies frontend and backend in one dev server:
- Single server at http://localhost:5173
- API routes at `/api/*` handled by Hono
- Hot reload works for both frontend and backend
- **Never** try to run separate servers or configure proxies

### API-Only Database Access
**Never** import Kysely or access D1 directly in frontend code. Always use HTTP APIs:
```typescript
// ✅ Correct - use API
const response = await fetch('/api/users');

// ❌ Wrong - direct database access
import { Kysely } from 'kysely';
```

## File Structure

```
src/
├── react-app/              # React frontend
│   ├── assets/             # Static assets (logos, images)
│   │   ├── logo-light.svg  # Logo for light theme
│   │   └── logo-dark.svg   # Logo for dark theme
│   ├── components/
│   │   ├── auth/           # Auth UI (LoginForm, ProtectedRoute, AuthOverlay)
│   │   ├── ui/             # shadcn/ui components
│   │   ├── TopBar.tsx      # Full-width header with logo and user menu
│   │   ├── Logo.tsx        # Theme-aware logo component
│   │   ├── ModeToggle.tsx  # Dark/light theme toggle
│   │   └── theme-provider.tsx  # Theme context with URL param support
│   ├── pages/              # Route pages
│   │   ├── auth/           # SignUp, VerifyEmail, ForgotPassword, ResetPassword
│   │   ├── Home.tsx        # Protected home
│   │   └── Profile.tsx     # User settings
│   └── lib/
│       └── auth/client.ts  # Better Auth React client
└── worker/                 # Hono backend
    ├── index.ts            # API routes + CLI endpoints
    ├── middleware/auth.ts  # Better Auth config
    ├── types/database.ts   # Kysely types
    └── utils/              # Email templates and provider adapters

src/cli/                    # CLI commands (auth user management + db utilities)
src/config.ts               # Global app configuration (app name, etc.)
bin/                        # CLI entry points (init, auth, db)
migrations/                 # D1 SQL migrations
seeds/                      # Idempotent SQL seed files
```

## Key Patterns

### Protected Routes
Wrap routes with `<ProtectedRoute>` to require authentication:
```tsx
<Route path="/dashboard" element={
  <ProtectedRoute>
    <Dashboard />
  </ProtectedRoute>
} />
```

### Adding API Endpoints
Add routes in `src/worker/index.ts`:
```typescript
// Public endpoint
app.get("/api/health", (c) => c.json({ status: "ok" }));

// Protected endpoint (requires auth)
app.use("/api/protected/*", authMiddleware);
app.get("/api/protected/data", async (c) => {
  const user = c.get("user");
  // ...
});
```

### Using shadcn/ui Components
Components are in `@/components/ui/`. Import directly:
```tsx
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
```

See `docs/SHADCN_COMPONENTS.md` for the full component list.

### Database Queries (Backend Only)
Use Kysely in worker code:
```typescript
import { Kysely } from "kysely";
import { D1Dialect } from "kysely-d1";
import type { Database } from "./types/database";

const db = new Kysely<Database>({
  dialect: new D1Dialect({ database: c.env.DB }),
});

const users = await db.selectFrom("users").selectAll().execute();
```

## Commands

> For remote database/auth commands, `.env.<env>` is loaded automatically — no manual `source` needed.

```bash
# Development
npm run dev                         # Start dev server (logs appear in terminal)

# Database — migrations
npm run db:migrate                  # Run migrations locally
npm run db:migrate:preview:safe     # Migrate preview with pre-migration backup
npm run db:migrate:production:safe  # Migrate production with pre-migration backup

# Database — migration status
npm run db:migrations:list
npm run db:migrations:list:preview
npm run db:migrations:list:production

# Database — backup & restore
npm run db:backup        # Export local D1 to .wrangler/backups/
npm run db:backup:production   # Export production D1 to .wrangler/backups/
npm run db:restore:latest        # Restore latest backup (auto-backs-up current DB first)
npm run db:restore -- --env production --file prod-backup.sql

# Database — time-travel (remote envs only)
npm run db:time-travel:info -- --env production
npm run db:time-travel:restore -- --env production --timestamp "2026-04-01 12:00:00"

# Database — seeds
npm run db:seed -- --env local --file admin-bootstrap.sql

# Auth CLI
# Requires CLI_API_KEY in .env.<env>; key must also be deployed to the worker
# dev server must be running for local
# Available user management commands (run `npm run auth help <command>` to see options):
npm run auth list-users

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [claudio-silva/cloudflare-fullstack-starter](https://github.com/claudio-silva/cloudflare-fullstack-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
