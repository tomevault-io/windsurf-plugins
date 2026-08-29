---
trigger: always_on
description: This file provides full context for Claude Code. Read it before making any changes.
---

# CLAUDE.md — Garage Boilerplate

This file provides full context for Claude Code. Read it before making any changes.
Client projects that fork this repo should update this file with their own project details.

---

## Project Overview

**Type:** Streamlined boilerplate for student capstone projects.
**Purpose:** Zero-friction foundation for Next.js + Firebase web applications. No Docker, no local emulator, no paid Firebase plan required — `pnpm install` plus a free Firebase project (Spark plan) is enough to run the app.

New to the repo? Read `docs/GUIDE.md` — it walks through building a feature end-to-end.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend framework | Next.js 16 (App Router, React 19) |
| Language | TypeScript 5 — strict mode |
| Styling | Tailwind CSS v4 (CSS-first config, no `tailwind.config.js`) |
| UI components | Raw Tailwind (shadcn can be added per project) |
| Backend | Firebase Cloud Functions v2 (Express fat-lambda) |
| Database | Firestore |
| Auth | Firebase Authentication |
| Package manager | pnpm workspaces — **always use pnpm, never npm or yarn** |
| Testing | Vitest + Testing Library (frontend) · Vitest + supertest (backend) |
| Git hooks | Lefthook (commit-msg: Conventional Commits · pre-commit: lint + format) |
| CI/CD | GitHub Actions |

---

## Repository Structure

```
/
├── frontend/          Next.js 16 App Router (deploys to Vercel)
├── backend/           Cloud Functions v2 Express fat-lambda
├── firebase/          Firestore rules, indexes
├── docs/              Architecture and conventions docs (start with GUIDE.md)
├── scripts/           Utility scripts (bootstrap, validate-placeholders, migrations)
└── .claude/           Claude Code harness (agents, skills, MCP, settings, hooks)
```

**Nested instructions** are loaded automatically when editing files in a package:
- `frontend/CLAUDE.md` — Next.js 16, App Router, Server Components, auth flow, design reference
- `backend/CLAUDE.md` — Express fat-lambda, route pattern, error handling, testing

---

## Codebase Map — read this instead of exploring

Everything a feature build needs already exists below. **Do not survey the codebase before implementing** — consult this map, then Read only the files you will edit. A complete worked example (every file of a real feature, verified) is in `docs/TUTORIAL-WALKTHROUGH.md`.

### Frontend building blocks

| File | Exports | Use for |
|------|---------|---------|
| `frontend/src/actions/auth.actions.ts` | `requireAuth()` (redirects if unauthed, returns session with `.uid`), `getServerSession()`, `serverSignOut()` | First line of every Server Action / protected page |
| `frontend/src/lib/firebase/admin.ts` | `adminAuth`, `adminDb` (lazy, `server-only`) | All server-side Firebase |
| `frontend/src/lib/firebase/client.ts` | `getClientApp/Auth/Db()` | Browser SDK (Client Components only) |
| `frontend/src/lib/firebase/firestore.ts` | `getUsersCollection()`, `userDoc(uid)` — add new collections here as `get{X}Collection()` functions (`typedCollection` is module-private) | Typed collection access |
| `frontend/src/lib/firebase/auth.ts` | `signInWithEmail`, `signUpWithEmail`, `signInWithGoogle`, `signOut`, `resetPassword`, `getIdToken` | Client sign-in flows |
| `frontend/src/hooks/useFirestore.ts` | `useCollection(ref, ...constraints)` → `{ data, loading, error }` (onSnapshot) | Realtime lists in Client Components |
| `frontend/src/hooks/useAuth.ts` | `useAuth()` → `{ user, profile, ... }` (AuthContext) | Current user in Client Components |
| `frontend/src/types/index.ts` | `ActionResult<T>` `{ success, error?, data? }` + re-exports of `types/auth.ts`, `types/firestore.ts` | Return type of every Server Action |
| `frontend/src/types/firestore.ts` | `UserProfile` — add new collection interfaces here (always with `_schemaVersion: 1`) | Collection types |
| `frontend/src/lib/validations/` | `loginSchema`, `signupSchema`, `registerSchema`, `resetPasswordSchema` (`auth.ts`) · `idSchema`, `paginationSchema` (`common.ts`) | Zod schemas — add feature schemas here or in the feature folder |
| `frontend/src/lib/utils.ts` | `cn()`, `formatDate`, `formatDatetime`, `truncate` | Class merging, formatting |
| `frontend/src/components/layout/` | `DashboardShell`, `Sidebar` (navItems array — add links here), `Navbar`, `PageHeader` | App shell |
| `frontend/src/components/shared/` | `ErrorBoundary`, `LoadingSpinner`, `FullPageSpinner`, `EmptyState { title, description?, icon?, action? }` | Loading/empty/error states |
| `frontend/src/app/api/auth/session/route.ts` | POST (token → `__session` cookie), DELETE | Already wired — don't touch for features |

### Backend building blocks

| File | Exports | Use for |
|------|---------|---------|
| `backend/src/app.ts` | `createApp({ verifyToken? })` | Composition; tests inject mock auth |
| `backend/src/middleware/auth.ts` | `AuthenticatedRequest` (`.user` = `AuthUser { uid, email, claims }`), `VerifyToken`, `verifyFirebaseToken` | Authed user in routes |
| `backend/src/lib/errors.ts` | `HttpError` + statics `badRequest/unauthorized/forbidden/notFound/conflict/internal` | All route errors, via `next(...)` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rishi-RMIT/a1-task1-boilerplate](https://github.com/Rishi-RMIT/a1-task1-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
