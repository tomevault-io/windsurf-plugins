---
trigger: always_on
description: Taru Villas is a **Next.js 16 survey management and quality assessment platform** for hotel property management. It features weighted scoring analytics, guest surveys, automatic task/issue tracking from low-score responses, and role-based access control.
---

# Taru Villas - Project Guide

## Overview

Taru Villas is a **Next.js 16 survey management and quality assessment platform** for hotel property management. It features weighted scoring analytics, guest surveys, automatic task/issue tracking from low-score responses, and role-based access control.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 16.1.6 (App Router, React 19) |
| Language | TypeScript 5 |
| Database | Supabase PostgreSQL via `postgres` (postgres.js) + Drizzle ORM |
| Auth | Supabase Auth (Google OAuth) |
| UI | shadcn/ui + Radix UI + Tailwind CSS 4 |
| Forms | React Hook Form + Zod v4 |
| Charts | Recharts |
| Tables | TanStack React Table |
| URL State | nuqs |
| Deployment | Vercel (serverless) |

## Project Structure

```
src/
├── app/
│   ├── (auth)/           # Login, OAuth callback (no auth required)
│   ├── (portal)/         # Authenticated routes
│   │   ├── admin/        # Admin-only pages (properties, users, templates, tasks)
│   │   ├── dashboard/    # Dashboard overview + property dashboards
│   │   ├── surveys/      # Survey list, creation, detail views
│   │   ├── tasks/        # Task list + detail (admin & PM)
│   │   └── settings/     # User settings
│   ├── (public)/         # Guest survey pages (token-based, no auth)
│   │   └── g/[token]/    # Guest survey form
│   └── api/              # API routes
│       ├── admin/guest-links/
│       ├── dashboard/
│       ├── properties/[id]/
│       ├── surveys/[id]/ + guest/
│       ├── tasks/[id]/
│       ├── templates/[id]/
│       └── users/[id]/
├── components/
│   ├── admin/            # Admin page client components
│   ├── dashboard/        # Dashboard overview + property charts
│   ├── layout/           # Sidebar, header
│   ├── surveys/          # Survey wizard, form, filters, score display
│   ├── tasks/            # Task list, detail
│   └── ui/               # shadcn/ui primitives
├── lib/
│   ├── auth/guards.ts    # requireAuth(), requireRole(), getProfile(), getUserProperties()
│   ├── db/
│   │   ├── index.ts      # Postgres connection (prepare: false for PgBouncer)
│   │   ├── schema.ts     # All tables, enums, relations
│   │   └── queries/      # Query functions by domain
│   │       ├── surveys.ts
│   │       ├── properties.ts
│   │       ├── profiles.ts
│   │       ├── tasks.ts
│   │       ├── guest-links.ts
│   │       └── dashboard.ts
│   ├── supabase/
│   │   ├── server.ts     # Server-side Supabase client
│   │   ├── admin.ts      # Service-role Supabase client
│   │   └── client.ts     # Browser Supabase client
│   └── utils.ts          # cn() helper (clsx + tailwind-merge)
├── middleware.ts          # Auth middleware (session check, public route exclusions)
drizzle/                   # Migration files
```

## Database Schema

### Enums
- `user_role`: admin | property_manager | staff
- `submission_status`: draft | submitted | reviewed
- `survey_type`: internal | guest
- `task_status`: open | investigating | closed

### Tables & Relationships

```
organizations (multi-tenant root)
├── profiles (auth users, FK to supabase auth.users)
│   └── propertyAssignments (M2M: user ↔ property)
├── properties
│   └── primaryPmId → profiles (default task assignee)
├── surveyTemplates
│   └── surveyCategories (weighted)
│       └── surveySubcategories
│           └── surveyQuestions (scale 1-10)
├── surveySubmissions
│   ├── surveyResponses (score + optional note + issueDescription)
│   └── guestSurveyLinks (token-based public access)
└── tasks (auto-created from low-score responses)
```

### Key Schema Details
- Survey templates have a **3-level hierarchy**: categories → subcategories → questions
- Categories have a `weight` field used for weighted average scoring
- Questions have configurable `scaleMin`/`scaleMax` (default 1-10)
- Submissions auto-generate slugs: `template-name-property-code-YYYY-MM-DD`
- Responses cascade-delete when submission is deleted
- Tasks are auto-created when response score <= 6 AND has issueDescription
- Tasks detect repeat issues (same question + property, previously closed)
- Guest links have unique constraint on (templateId, propertyId)

## Authentication & Authorization

### Auth Flow
1. Google OAuth via Supabase Auth
2. Auto-provisioning on first login (first user = admin, rest = staff)
3. Users must have @taruvillas.com email
4. Middleware checks session on every request

### Public Routes (no auth)
- `/login`, `/callback`
- `/g/*` (guest surveys)
- `/api/surveys/guest`
- `/api/cron/*` (uses Bearer CRON_SECRET)

### Role Permissions

| Route/Feature | admin | property_manager | staff |
|---------------|-------|-------------------|-------|
| `/dashboard` overview | Full access | Redirect to /surveys | Redirect to /surveys |
| `/dashboard/[propertyId]` | All properties | Assigned properties only | Redirect to /surveys |
| `/surveys` (list/create) | All | All | All |
| `/admin/*` pages | Full CRUD | No access | No access |
| `/tasks` | All org tasks | Assigned property tasks | No access (403) |
| Property/User CRUD APIs | Full | Read only | Read only |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sonaljay/Taru-Villas](https://github.com/sonaljay/Taru-Villas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
