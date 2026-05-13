---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

MoonCakeTV (月饼TV) is a **simplified** Next.js 15 web application for video streaming aggregation and search. This is one of several independent repositories in the MoonCake TV multi-repository workspace.

**Architecture Philosophy:**

- **Single user mode** - No multi-user complexity
- **Optional password** - Public access or password-protected
- **File-based storage** - No database required
- **VPS-first** - Simple deployment without Docker
- **Minimal dependencies** - Only what's necessary

**Tech Stack:**

- Next.js 15 with App Router and React 19
- TypeScript 5.x
- Tailwind CSS 4.x
- **File-based JSON storage** (no database)
- JWT authentication with `jose`
- bcryptjs for password hashing
- Radix UI components with shadcn/ui
- Vidstack/HLS.js for video playback
- Zustand for state management
- Zod v4 for validation

**Node.js Requirement:** >=22.0.0

## TypeScript Configuration

**Path Aliases:**

- `@/*` maps to `./src/*` (e.g., `@/components`, `@/lib`)
- `~/*` maps to `./public/*` (e.g., `~/logo.png`)

Always use these aliases instead of relative imports for better maintainability.

## Common Development Commands

```bash
# Development
npm install              # Install dependencies
npm run dev              # Start dev server on 0.0.0.0:3333
npm run build            # Production build
npm run start            # Start production server

# Code Quality
npm run lint             # Run ESLint
npm run lint:fix         # Fix ESLint errors and format code
npm run typecheck        # TypeScript type checking
npm run format           # Format code with Prettier
npm run format:check     # Check formatting without changes

# Testing
npm run test             # Run Jest tests
npm run test:watch       # Run tests in watch mode

# Git Deployment
make origin              # Push to origin remote
make tea                 # Push to tea remote
make vercel              # Push to vercel remote

# Simple Commands
make dev                 # npm run dev
make build               # npm run build
```

## Authentication Architecture

The application uses a **simplified single-user authentication** system with optional password protection.

### How It Works

**Two modes:**

1. **No password set** → Anyone can access (public mode)
2. **Password set** → Login required for all protected routes

**Middleware flow (`src/middleware.ts`):**

1. Check if password has been set (via `isPasswordSet()`)
2. If no password → Allow access (skip authentication)
3. If password exists → Require `mc-auth-token` JWT cookie
4. Redirect to `/login` if not authenticated

**Protected vs Public paths:**

- Public: `/_next`, `/favicon.ico`, `/login`, `/api/login`, `/api/logout`, `/api/server-config`, static files
- Protected: All other routes (when password is set)

### Setting a Password

Visit `/login` page:

- If no password is set, you can set one
- Password is hashed with bcryptjs and stored in `data/user-data.json`
- Once set, the app requires login for all protected routes

### Removing Password Protection

Delete or edit `data/user-data.json` and remove the `password_hash` value.

**Warning:** This will also delete bookmarks and watch history unless you backup the file first.

## Data Storage

All user data is stored in a single JSON file: `data/user-data.json`

**File structure:**

```json
{
  "password_hash": "",
  "bookmarks": [
    {
      "id": "video_123",
      "title": "电影名称",
      "thumbnail": "https://...",
      "url": "https://...",
      "added_at": "2025-01-15T10:30:00.000Z"
    }
  ],
  "watch_history": [
    {
      "id": "video_123",
      "title": "电影名称",
      "progress": 120,
      "last_watched": "2025-01-15T10:30:00.000Z"
    }
  ],
  "settings": {}
}
```

### Data Management

**Backup:**

```bash
cp data/user-data.json backup/user-data-$(date +%Y%m%d).json
```

**Restore:**

```bash
cp backup/user-data-20250115.json data/user-data.json
```

**Migration to new server:**

```bash
# Old server
scp data/user-data.json user@new-server:/path/to/mooncaketv-web/data/

# New server - data is ready!
```

**Reset everything:**

```bash
rm data/user-data.json
# File will be recreated with defaults on next startup
```

## Project Structure

### Key Directories

- **`data/`** - Data storage directory
  - `user-data.json` - All user data (auto-created)

- **`src/app/`** - Next.js App Router pages and layouts
  - `api/` - API route handlers
    - `login/` - User login endpoint
    - `logout/` - User logout endpoint
    - `bookmarks/` - Bookmarks CRUD API
    - `history/` - Watch history API
    - `server-config/` - Server configuration endpoint
    - `image-proxy/` - Image proxy for CORS handling
    - `speed-test/` - Network speed testing endpoint
  - `play/` - Video playback page
  - `search/` - Search results page
  - `bookmarks/` - User bookmarks page
  - `watch-history/` - Watch history page
  - `settings/` - User settings page
  - `login/` - Login page

- **`src/components/`** - React components
  - `ui/` - shadcn/ui components (Radix UI primitives)
  - `common/` - Shared common components
  - `mc-play/` - Video player components (Vidstack-based)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MoonCakeTV/MoonCakeTV](https://github.com/MoonCakeTV/MoonCakeTV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
