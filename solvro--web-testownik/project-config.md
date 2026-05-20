---
trigger: always_on
description: A quiz/study platform for Wrocław University of Technology students, built by KN Solvro. Next.js 16 App Router + React
---

# Copilot Instructions — Testownik Solvro Frontend

## Project Overview

A quiz/study platform for Wrocław University of Technology students, built by KN Solvro. Next.js 16 App Router + React
19 + TanStack Query v5 + Tailwind CSS v4 + shadcn/ui (Radix). Backend is a separate Django REST API (
`Solvro/backend-testownik`). UI strings are in **Polish**; code identifiers are in **English**.

## Architecture

### Page Pattern (SSR ↔ Client split)

Every route follows a two-file convention in `src/app/<route>/`:

- **`page.tsx`** — Server Component. Reads cookies via `next/headers`, creates a `QueryClient`, prefetches data through
  service classes, wraps content in `<HydrationBoundary>`. Exports `metadata` (Polish titles).
- **`client.tsx`** — Client Component (`"use client"`). Named export `XxxPageClient`. Consumes hydrated queries and
  accesses `AppContext` for auth/services.

Reference: `src/app/quizzes/page.tsx` → `src/app/quizzes/client.tsx`.

Some simpler pages (e.g. `grades`, `create-quiz`) skip SSR prefetching and render the client component directly.

### Service Layer (`src/services/`)

Class-based API services extending `BaseApiService`. Accessed via singleton `ServiceRegistry` exposed through React
Context (`AppContext`). Three services: `QuizService`, `UserService`, `ImageService`.

Access pattern in client components:

```ts
const { services, isGuest } = useContext(AppContext);
const quizzes = await services.quiz.getQuizzes();
```

### Auth

JWT (HS256) stored in cookies (`access_token`, `refresh_token`). Guest mode uses `is_guest` cookie. Client-side decoding
via `jose.decodeJwt()` (no verification). Server-side verification via `jose.jwtVerify()` with `JWT_SECRET` env var.
Auth context is provided by `AppContextProvider` in `src/app-context-provider.tsx`.

### State Management & Data Fetching

**TanStack Query v5** exclusively (no Redux/Zustand). Single `QueryClientProvider` at app root (
`src/app/providers.tsx`).

**SSR-first pattern:** Prefer server-side data prefetch + hydration over client-only fetches. A typical page:

1. Server (`page.tsx`) calls `queryClient.prefetchQuery()` with the service
2. Dehydrates state: `<HydrationBoundary state={dehydrate(queryClient)}>`
3. Client reads hydrated cache immediately (no duplicate fetch)

Example: `src/app/quizzes/page.tsx` prefetches `["user-quizzes", isGuest]` — client hook in `useQuizzes()` uses the same
key.

**Query keys** include `isGuest` flag for cache segmentation. Default `staleTime` is 60s (`src/lib/query-client.ts`).
Keep keys consistent between SSR prefetch and client hooks.

### Middleware & Auth Guards

`src/proxy.ts` (Next.js middleware) enforces auth on protected routes (`/profile`, `/quizzes`, `/grades`,
`/create-quiz`, `/edit-quiz`, `/import-quiz`). Handles:

- Token validation and refresh (expires with 30s buffer)
- Automatic token refresh using `POST /token/refresh/`
- Fallback to login redirect if no valid token
- Cookie forwarding from backend responses

### Env Variables

Validated with `@t3-oss/env-nextjs` + Zod in `src/env.ts`. Required: `NEXT_PUBLIC_API_URL`. Optional:
`NEXT_PUBLIC_TURN_USERNAME`, `NEXT_PUBLIC_TURN_CREDENTIAL` (TURN relay for P2P), `JWT_SECRET` (server),
`INTERNAL_API_KEY` (server).

## Key Conventions

- **Path alias:** `@/` maps to `src/` (tsconfig paths)
- **UI components:** shadcn/ui in `src/components/ui/`, project components alongside in `src/components/`
- **Hooks:** `src/hooks/use-{feature}.ts` (kebab-case). Feature-specific hooks may be colocated (e.g.
  `src/components/quiz/hooks/`)
- **Validation:** Zod v4 schemas in `src/lib/schemas/`

### Commit Format

Use **Conventional Commits** via `@solvro/config` commitlint enforcement.

**Format:** `<type>(optional scope): present-tense description in English`

**Types:** `feat`, `fix`, `refactor`, `chore`, `docs`, `ci`, `test`, `build`, `release`

**Examples:**

- `feat(quiz): add cross-device quiz continuity`
- `fix(auth): correct token refresh timeout`
- `refactor(editor): simplify question form`
- `test(hooks): add useQuizzes integration tests`

### Branch Naming

**Format:** `<prefix>/<issue>-short-description`

**Prefixes:** `feat/`, `fix/`, `hotfix/`, `design/`, `refactor/`, `test/`, `docs/`

**Examples:**

- `feat/123-add-solvro-auth`
- `fix/87-fix-date-display`
- `refactor/210-quiz-import-logic`

### Code Style

- **Formatting:** Prettier via `@solvro/config/prettier` — auto-run on commit via husky + lint-staged
- **Linting:** ESLint flat config via `@solvro/config/eslint` (see `eslint.config.js`)

## Commands

| Task       | Command                                    |
| ---------- | ------------------------------------------ |
| Dev server | `pnpm run dev`                             |
| Build      | `pnpm run build`                           |
| Lint       | `pnpm run lint` / `npm run lint:fix`       |
| Format     | `pnpm run format` / `npm run format:check` |
| Type check | `pnpm run typecheck`                       |
| Tests      | `pnpm run test` (Vitest)                   |

## Cross-Device Quiz Continuity (PeerJS)

Users can sync quiz progress across devices (desktop, tablet, mobile) via WebRTC peer-to-peer.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Solvro/web-testownik](https://github.com/Solvro/web-testownik) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
