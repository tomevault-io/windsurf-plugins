---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🚨 CONSTITUTIONAL REQUIREMENTS (NON-NEGOTIABLE)

This project follows the FTC TeamForge Constitution (see `.specify/memory/constitution.md`). These principles MUST be followed:

### Component-Based Architecture

- ALL UI features must be built as reusable React components using shadcn/ui patterns
- Components must be self-contained, independently testable, and documented with TypeScript interfaces
- No direct DOM manipulation outside of component boundaries

### Test-First Development (TDD Mandatory)

- Tests MUST be written FIRST → User approved → Tests fail → Then implement
- Red-Green-Refactor cycle strictly enforced
- Focus on user behavior testing over implementation details
- Every user interaction must have test coverage BEFORE implementation

### Database Migration Management

- ALL database schema changes MUST use incremental migration scripts in `/database/migrations/`
- Files MUST be numbered sequentially (001_initial.sql, 002_add_users.sql, etc.)
- Each migration MUST include both UP and DOWN operations
- MUST be idempotent and include RLS policy updates
- NO direct database schema modifications allowed
- **CRITICAL**: Any database changes that work in development/production MUST be immediately reflected in the `001_initial_schema_fixed.sql` migration script
- This ensures new users running FRE get the latest working schema and policies
- Database fixes applied via Supabase console MUST be backported to migration files

### Consistent Navigation Experience

- Sidebar navigation MUST be available on every authenticated page
- ALL authenticated pages MUST use `DashboardLayout` component
- Users must navigate between features without losing context

### Education-First Development

- Primary users are students (ages 12-18) learning robotics
- Features must be intuitive with help text and guided experiences
- Complex workflows need documentation and learning opportunities

## Development Commands

### Core Commands

- `npm run dev` - Start development server with Turbopack
- `npm run build` - Build production application with Turbopack
- `npm run start` - Start production server
- `npm run lint` - Run ESLint for code quality checks

### TypeScript and Type Checking

- Project uses TypeScript with strict mode enabled
- Run TypeScript compiler check: `npx tsc --noEmit`
- Path aliases configured: `@/*` maps to `./src/*`

## Architecture Overview

### Application Structure

This is a **Next.js 15** application called **FTC TeamForge** for FTC robotics team management with **Supabase** backend.

#### Key Architectural Patterns

- **Context-based State Management**: Uses React Context for auth (`AuthProvider`) and app data (`AppDataProvider`)
- **Season-based Data Model**: All data is organized by seasons with multi-season support
- **Team-based Access Control**: Row Level Security (RLS) ensures teams only access their own data
- **Component-First Architecture**: Heavy use of shadcn/ui components with Tailwind CSS v4

#### Core Context Providers (src/components/)

- `AuthProvider.tsx` - Supabase authentication state management
- `AppDataProvider.tsx` - Team data, members, and current season context
- `ProtectedRoute.tsx` - Route protection wrapper
- `QueryProvider.tsx` - TanStack Query (React Query) configuration with IndexedDB persistence
- `NotebookProvider.tsx` - Notebook state management and auto-save functionality
- `ThemeProvider.tsx` - Theme and dark mode management
- `AccentColorProvider.tsx` - User-selected accent color theming

#### Database Integration (src/lib/)

- `supabase.ts` - Supabase client and database status checking
- `checkDatabaseStatus()` function handles initial setup verification
- `api-auth.ts` - API route authentication helpers
- `api-errors.ts` - Standardized API error responses
- `rateLimit.ts` - Rate limiting for API endpoints (uses Upstash Redis-like storage)
- `queryClient.ts` - TanStack Query client configuration with offline persistence

### Database Architecture

- **PostgreSQL** with Supabase backend
- **Row Level Security (RLS)** policies on all tables
- **Season-based data isolation** - all content tied to specific seasons
- **Team-based access control** - users can only access their team's data

#### Core Tables

- `teams` - Team information (number, name, school)
- `seasons` - Season management with current season tracking
- `team_members` - User-team relationships with roles
- `users` - User profiles and authentication data

### First Run Experience (FRE)

The app includes a comprehensive setup flow:

1. **Database Setup** - Initializes all required tables and functions
2. **Team Setup** - Creates team and admin user
3. **Season Setup** - Creates initial season configuration

Located in: `src/components/FirstRunExperience.tsx`, `src/components/DatabaseSetup.tsx`

### Component Organization

#### UI Components (src/components/ui/)

- Uses **shadcn/ui** component library
- **Tailwind CSS v4** for styling
- **Radix UI** primitives for accessibility

#### Feature Components

- `DashboardLayout.tsx` - Main application layout with sidebar navigation (REQUIRED for all auth pages)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [incredibotsftc/teamforge](https://github.com/incredibotsftc/teamforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
