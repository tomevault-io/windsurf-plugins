---
trigger: always_on
description: Guidance for coding agents working in this repository. Read it fully before making changes.
---

# AGENTS.md

Guidance for coding agents working in this repository. Read it fully before making changes.

This file follows the [AGENTS.md](https://agents.md) convention and is the single
source of truth; `CLAUDE.md` is a pointer to it. Keep it that way — this document
was previously duplicated, the two copies drifted, and a scrub applied to one of
them left the other still naming a production host.

---

## What is Backenly?

**Backenly** (live at https://backenly.com) is an **autonomous backend platform** that turns product descriptions into running backend infrastructure. It plans the backend, applies the infrastructure, verifies the runtime, and keeps every change reviewable and reversible. Developers describe their backend in natural language and Backenly automatically creates database tables, REST APIs, auth systems, storage, and more — no manual backend code required. (Category note: Backenly is **not** an "AI BaaS." It does not just generate resources — it manages backend change safely.)

**Tech Stack:** Next.js 14, React 18, Express.js, PostgreSQL (Prisma), MongoDB (optional), OpenAI API, Paddle payments, TailwindCSS, TypeScript throughout.

**GitHub Repo:** https://github.com/backenly/backenly

---

## Critical Concepts (Read Before Touching Anything)

### 1. Two Completely Separate User Types

| Type | Who | Auth System | Where Data Lives |
|------|-----|-------------|-----------------|
| **Platform users** | Developers using backenly.com | `lib/auth/` + `JWT_SECRET` | `public` schema (Prisma models) |
| **End users** | Users of a project built *on* Backenly | `app/api/v1/[projectId]/auth/` + `project.jwtSecret` | `workspace_{projectId}.users` table |

**Never mix these up.** Platform auth touches `lib/auth/middleware.ts`. End-user auth uses project-scoped JWTs. They are completely isolated.

### 2. Multi-Tenant Schema Isolation

Each project gets its own PostgreSQL schema: `workspace_{projectId}`.

- All end-user tables, rows, and auth live inside that schema.
- The platform's own data lives in the `public` schema (managed by Prisma).
- Key files: `lib/tenant/isolation.ts`, `lib/services/workspaceDatabase.ts`

### 3. Feature Placement Rule (Do Not Violate)

From `lib/config/QUICK_REFERENCE.ts`:

> Only the **Database Management** section creates new backend reality (tables, APIs). All other sections (monitoring, auth settings, billing) manage *existing* reality.

**Never** add "quick create" or "AI generate" buttons outside the Database/AI chat flow.

---

## Project Structure

```
/
├── app/                          # Next.js app directory
│   ├── api/                      # ~234 API route handlers
│   │   ├── ai/chat/route.ts      # Main AI chat entry point
│   │   ├── ai-workspace/         # AI plan/apply/diff/detect routes
│   │   ├── auth/                 # Platform auth (login, OAuth, JWT)
│   │   ├── billing/              # Paddle subscription + AI usage
│   │   ├── database/             # Table & schema management
│   │   ├── database-brain/       # AI-powered DB analysis & fixes
│   │   ├── deployments/          # Deploy pipeline, logs, rollback
│   │   ├── monitoring/           # Metrics, anomalies, incidents
│   │   ├── storage/              # File storage
│   │   ├── cron/                 # Scheduled jobs (archive, cleanup, billing)
│   │   └── v1/[projectId]/       # PUBLIC runtime API for end-users
│   │       ├── auth/             # End-user sign-up/sign-in
│   │       ├── db/               # CRUD for workspace tables
│   │       ├── storage/          # File operations
│   │       ├── realtime/         # SSE + PostgreSQL LISTEN/NOTIFY
│   │       ├── broadcast/        # Ephemeral pub/sub
│   │       └── triggers/         # Event trigger management
│   └── app/                      # UI pages (Next.js app router)
│       ├── projects/[id]/        # Per-project dashboard
│       │   ├── (inspector)/      # Main IDE-like view
│       │   │   ├── database/     # DB management + AI chat
│       │   │   ├── api-builder/  # Visual API editor
│       │   │   ├── auth/         # Auth config
│       │   │   ├── storage/      # File storage UI
│       │   │   ├── users/        # End-user management
│       │   │   ├── monitoring/   # Metrics UI
│       │   │   ├── realtime/     # Realtime UI
│       │   │   ├── functions/    # Serverless AI functions
│       │   │   └── deploy/       # Deployment UI
│       │   ├── settings/         # Project settings
│       │   ├── history/          # Change history
│       │   └── iam/              # IAM config
│       ├── settings/             # Account settings
│       └── pricing/              # Pricing page
├── lib/                          # All shared business logic
│   ├── ai/                       # AI orchestration (THE CORE — see below)
│   ├── auth/                     # Platform auth & RBAC
│   ├── db/                       # DB clients (Prisma, MongoDB, hybrid)
│   ├── billing/                  # Credits, quotas, grace periods
│   ├── deployment/               # Deployment pipeline
│   ├── tenant/                   # Multi-tenancy isolation
│   ├── services/                 # Workspace DB, triggers, RLS, etc.
│   ├── middleware/               # Express middleware (20+ files)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [backenly/backenly](https://github.com/backenly/backenly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
