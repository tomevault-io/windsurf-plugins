---
trigger: always_on
description: Goodreads alternative built on Bluesky's AT Protocol. Server-rendered Hono JSX with minimal client-side hydration via `hono/jsx/dom`. Bun runtime, SQLite via Kysely, Tailwind CSS v4.
---

# BookHive — Agent Reference Index

Goodreads alternative built on Bluesky's AT Protocol. Server-rendered Hono JSX with minimal client-side hydration via `hono/jsx/dom`. Bun runtime, SQLite via Kysely, Tailwind CSS v4.

## User preferences (do not remove this)

- Do not commit any changes to git unless otherwise instructed

## Input Validation

Routes must always validate inputs using either:

- **zValidator** (with Zod) for standard Hono routes — use `zValidator("query", schema)`, `zValidator("json", schema)`, or `zValidator("form", schema)` as appropriate.
- **XRPC router validators** for AT Protocol XRPC endpoints in `src/xrpc/`. Add lexicons & regenerate the generated types with `bun run lexgen`

## Architecture at a Glance

```
Browser ──> Bun.serve() ──> Hono app ──> Server-rendered JSX pages
                │                             │
                │ /_bundle (HTML import)       ├── SQLite (Kysely ORM)
                │   └── entry.html             ├── KV cache (unstorage + SQLite)
                │       ├── src/client/index.tsx  ├── Bluesky PDS (ATProto writes)
                │       └── src/index.css      └── Goodreads scraper
                │
                └── static files (public/)
```

**Key pattern**: Server components (`src/pages/`) render full HTML. Only 4 islands are hydrated client-side (`src/client/`). Most interactivity is CSS-only (peer/checked selectors for tabs, dropdowns, modals) or inline `<Script>` vanilla JS.

## Entry Points

| File                   | Purpose                                            |
| ---------------------- | -------------------------------------------------- |
| `src/index.ts`         | Bun.serve — HTML bundle route + Hono fetch handler |
| `src/server.ts`        | Creates app dependencies and Hono app              |
| `src/app.ts`           | Hono app factory — all middleware + route mounting |
| `src/entry.html`       | Bun HTML bundle entry (imports CSS + client JS)    |
| `src/client/index.tsx` | Client bundle entry — mounts 4 hydrated components |

## Routes

All routes composed in `src/routes/main.tsx` → `src/app.ts`.

- `/` → `src/pages/home.tsx` — landing, hero, book list, buzzes (redirects to `/home` when logged in)
- `/home` → `src/pages/home.tsx` — authenticated home
- `/search` → `src/pages/searchResults.tsx` — search results page
- `/feed` → `src/pages/feed.tsx` — activity feed, friends/all/tracking tabs
- `/app` → `src/pages/app.tsx` — iOS app landing
- `/privacy-policy` → `src/pages/privacy-policy.tsx` — privacy policy
- `/legal` → `src/pages/terms.tsx` — terms of service
- `/pds` → `src/pages/pds.tsx` — PDS info page
- `/import` → `src/pages/import.tsx` — Goodreads/StoryGraph CSV import, SSE progress
- `/explore` → `src/pages/explore.tsx` — explore hub
- `/explore/genres` → `src/pages/genres.tsx` — genre directory
- `/explore/genres/:genre` → `src/pages/genreBooks.tsx` — books by genre, paginated, sortable
- `/explore/authors` → `src/pages/authorDirectory.tsx` — author directory
- `/authors/:author` → `src/pages/authorBooks.tsx` — books by author, paginated
- `/genres`, `/genres/:genre` → legacy redirects to `/explore/genres`
- `/profile`, `/profile/:handle` → `src/pages/profile.tsx` — user profile, book shelves
- `/profile/:handle/stats` → `src/pages/readingStats.tsx` — reading stats by year
- `/refresh-books` → `src/routes/profile.tsx` — re-sync books from PDS
- `/books/:hiveId` → `src/pages/bookInfo.tsx` — book detail, status, rating, review, progress
- `/books/:hiveId/comments` → `src/pages/comments.tsx` — comments/reviews section
- `/comments` (POST/DELETE) → `src/routes/comments.tsx` — comment mutations
- `/api/update-book`, `/api/update-comment`, `/api/follow`, `/api/follow-form` → `src/routes/api.tsx` — JSON API
- `/login`, `/logout`, `/oauth/callback` → `src/auth/router.tsx` — OAuth auth flows
- `/import` (POST /goodreads, /storygraph) → `src/routes/import.ts` — CSV import handler
- `/admin/export` → `src/routes/admin.ts` — DB export
- `/xrpc/*` → `src/xrpc/router.ts` — AT Protocol XRPC endpoints
- `/images/*` → `src/routes/main.tsx` — IPX image proxy/transform
- `/.well-known/atproto-did` → returns DID constant

Shared route helpers: `src/routes/lib.ts` (searchBooks, refetchBooks, refetchBuzzes, syncFollows).

## Server-Side Pages (`src/pages/`)

Each file exports a Hono JSX component rendered server-side.

| File                  | Renders                                                            |
| --------------------- | ------------------------------------------------------------------ |
| `layout.tsx`          | HTML shell — meta tags, asset injection, `<head>`/`<body>` wrapper |
| `navbar.tsx`          | Top nav bar with user menu, search mount point                     |
| `simple-navbar.tsx`   | Simplified nav bar variant                                         |
| `sidebar.tsx`         | Sidebar layout component                                           |
| `home.tsx`            | Landing/home page — hero, features, book list, buzzes              |
| `marketing.tsx`       | Marketing landing page for logged-out users                        |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nperez0111/bookhive](https://github.com/nperez0111/bookhive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
