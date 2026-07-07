---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Journal App** - A full-stack personal journaling application using the B-MAD methodology. The app features a guided 4-section entry flow (Professional, Personal, Learning, Gratitude) with AI-powered coaching and pattern recognition. Built with React 19, Node.js/Express, PostgreSQL, and Anthropic Claude API.

**B-MAD Artifacts**: All planning, architecture, and design documents are in `_bmad-output/` directory.

## Repository Structure

```
.
├── client/              # React 19 frontend (Vite + Tailwind)
├── server/              # Node.js backend (Express + Better Auth + Drizzle ORM)
├── scripts/             # Infrastructure validation (Deployment scripts)
├── _bmad-output/        # B-MAD artifacts (PRD, Architecture, UX Design, Epics)
├── docker-compose.yml   # Local development environment
└── package.json         # Root workspace configuration
```

## Common Commands

### Root Level
```bash
npm install                    # Install all dependencies (client + server + root)
npm run test:all              # Run all tests (infrastructure + server)
npm run test:server           # Run server tests only
npm run test:infra            # Run infrastructure validation script
npm run lint:client           # Lint client code
```

### Client (`client/`)
```bash
npm run dev                   # Start Vite dev server (http://localhost:5173)
npm run build                 # Build for production
npm run preview              # Preview production build locally
npm run lint                 # Run ESLint
```

### Server (`server/`)
```bash
npm run dev                  # Start Express dev server with tsx watch
npm run build                # Compile TypeScript to dist/
npm start                    # Run compiled server
npm test                     # Run Vitest test suite
npm run db:migrate           # Run Drizzle migrations
npm run db:rls               # Apply Row-Level Security policies to database
npm run db:generate          # Generate migration files from schema changes
npm run seed:user            # Create test user in database
npm run test:rls             # Test RLS policies
```

### Docker
```bash
docker-compose up -d --build     # Start full stack (server + client + postgres)
docker-compose down              # Stop all services
docker-compose logs -f           # View logs from all services
```

## Setup Instructions

1. **Install dependencies**: `npm install` (installs client, server, and root deps)

2. **Configure environment variables**: Create `.env` in root directory:
   ```
   DATABASE_URL=postgresql://app_user:password@localhost:5432/journal
   DATABASE_ADMIN_URL=postgresql://postgres:password@localhost:5432/journal
   BETTER_AUTH_SECRET=your-secret-key
   ANTHROPIC_API_KEY=your-api-key
   ```

3. **Start Docker environment**: `docker-compose up -d --build`
   - PostgreSQL runs on localhost:5432
   - Server runs on localhost:3000
   - Client dev server runs on localhost:5173

4. **Apply database migrations and RLS**:
   ```bash
   npm run --prefix server db:migrate
   npm run --prefix server db:rls
   ```

5. **Seed test data** (optional): `npm run --prefix server seed:user`

6. **Access the app**: Visit http://localhost:5173

## Architecture

### Application Flow
1. **Frontend** - React router in `client/src/App.jsx` manages: LoginView → HomeView ↔ EntryCreationView
2. **Backend** - Express server with Better Auth and RLS-protected database queries
3. **Database** - PostgreSQL with Row-Level Security policies enforcing data isolation

### Key Patterns

**Frontend Architecture**
- **React Context** for auth state (useAuth hook from AuthContext)
- **View-based components** for full-page flows (LoginView, HomeView, EntryCreationView)
- **Service layer** in `src/services/` abstracts all API calls to backend
- **Card-stack UI** for 4-section entry creation with Framer Motion animations

**Backend Architecture**
- **Better Auth** for authentication (email/password signup/signin)
- **Drizzle ORM** for type-safe database queries
- **Row-Level Security (RLS)** on PostgreSQL tables enforces user data isolation
- **Express middleware** for CORS, error handling, and auth verification

**Database Design**
- **user_profiles** - User metadata (streaks, entry count, onboarding status)
- **entries** - Journal entries with 4 text sections + metadata
  - Unique constraint on `(user_id, entry_date)` - one entry per day max
  - Auto-saves use upsert pattern via `saveEntry()` service
  - RLS policies: Users can only read/write their own entries

### Database Schema

**Key Tables:**
- `user_profiles` - User metadata and tracking
- `entries` - Journal entries with 4 sections (professional_recap, personal_recap, learning_reflections, gratitude)
- `sessions` - Better Auth session management
- `accounts` - Better Auth account linking

**Critical Fields on entries:**
- `entry_date` (DATE) - ISO format `YYYY-MM-DD`, defaults to TODAY()
- `completed` (BOOLEAN) - True when all 4 sections filled
- `created_at`, `updated_at` (TIMESTAMP) - Auto-managed by database
- `user_id` (UUID) - Foreign key, enforced by RLS

### Client Services Layer

**authService.js**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Philippe-arnd/JournalVibecoded](https://github.com/Philippe-arnd/JournalVibecoded) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
