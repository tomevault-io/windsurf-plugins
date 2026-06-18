---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Local Configuration

Create `./CLAUDE.local.md` for personal settings that won't be committed:

```markdown
# Example CLAUDE.local.md

## GitHub CLI

Always use the `your-username` account for gh commands.

## Sentry CLI

Use org `your-org` and project `your-project`.

## Personal Notes

- My test user ID: user_abc123
- Local dev URL: http://localhost:3000
```

**Use cases for `CLAUDE.local.md`:**

- CLI tool credentials (GitHub, Sentry, Fly.io accounts)
- Personal test data and user IDs
- Local environment URLs and ports
- Workflow preferences specific to you
- Notes that shouldn't affect other contributors

---

## ADHX

**Save now. Read never. Find always.**

A Twitter/X bookmark manager for people who bookmark everything and read nothing. Built with Next.js 16. Also previews and saves Instagram Reels, TikTok videos, and YouTube Shorts via the same URL-prefix trick (TikTok/Reels offer MP4 download; YouTube plays via the official iframe embed).

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                  BROWSER                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│  Landing Page          Main Feed              URL Prefix Feature            │
│  ┌─────────────┐      ┌─────────────┐        ┌─────────────────────┐       │
│  │ /           │      │ / (auth'd)  │        │ /{user}/status/{id} │       │
│  │ Marketing   │      │ FeedGrid    │        │ Quick-save tweet    │       │
│  │ OAuth Start │      │ Lightbox    │        │ → Add & redirect    │       │
│  └─────────────┘      │ FilterBar   │        └─────────────────────┘       │
│                       └─────────────┘                                       │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              NEXT.JS API ROUTES                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Auth                    Data                      Media                     │
│  ┌──────────────┐       ┌──────────────┐         ┌──────────────┐          │
│  │ /auth/twitter│       │ /api/feed    │         │ /api/media/  │          │
│  │ /auth/callback       │ /api/sync    │◄──SSE   │   video      │          │
│  │ /auth/status │       │ /api/tweets/ │         └──────┬───────┘          │
│  └──────┬───────┘       │   add        │                │                   │
│         │               │ /api/bookmarks               │                   │
│         │               └──────┬───────┘                │                   │
│         │                      │                        │                   │
└─────────┼──────────────────────┼────────────────────────┼───────────────────┘
          │                      │                        │
          ▼                      ▼                        ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────────┐
│   Twitter API   │    │  SQLite + Drizzle│    │    FxTwitter API    │
│   (OAuth 2.0)   │    │                  │    │ (Media proxy/embed) │
│                 │    │  bookmarks       │    │                     │
│  • Auth tokens  │    │  bookmark_media  │    │  • Video URLs       │
│  • User info    │    │  bookmark_tags   │    │  • Photo URLs       │
│  • Bookmarks    │    │  read_status     │    │  • Tweet enrichment │
│                 │    │  sync_logs       │    │                     │
└─────────────────┘    └─────────────────┘    └─────────────────────┘
```

**Data Flow:**

1. **Sync**: Twitter API → Process tweets → SQLite (via `/api/sync` SSE stream)
2. **Add**: URL → FxTwitter enrichment → SQLite (via `/api/tweets/add`)
3. **View**: SQLite → Feed API → React components
4. **Media**: FxTwitter proxy → Video/Photo display (bypasses Twitter CORS)

## Quick Start

```bash
pnpm install
pnpm dev         # Start dev server at localhost:3001
pnpm build       # Production build
pnpm test        # Run all 872 tests
```

## Tech Stack

- **Framework**: Next.js 16 (App Router) + React 19
- **Database**: SQLite via better-sqlite3 + Drizzle ORM 0.45
- **Styling**: Tailwind CSS 3.4 + clsx + tailwind-merge
- **Twitter API**: twitter-api-v2 with OAuth 2.0 PKCE
- **Auth**: JWT-signed session cookies (jose)
- **Monitoring**: Sentry (error tracking + metrics)
- **Icons**: lucide-react
- **Fonts**: Indie Flower (brand), IBM Plex Sans/Inter/Lexend/Atkinson Hyperlegible (body - user selectable)
- **Testing**: Vitest
- **Deployment**: Fly.io with automated deploys via GitHub Actions

## Security

### Session Management

Sessions use JWT signing via `jose` library to prevent tampering:

- Cookie name: `adhx_session`
- Signed with `SESSION_SECRET` or `TWITTER_CLIENT_SECRET`
- 30-day expiration
- httpOnly, secure (in production), sameSite: lax

### Authentication


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [itsmemeworks/adhx](https://github.com/itsmemeworks/adhx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
