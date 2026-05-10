---
trigger: always_on
description: Web app displaying translated Bilibili video comments for Ludwig's "Tip to Tip" motorcycle series across China. Translates Chinese audience reactions into English with cultural annotations so international viewers can follow along.
---

# t2t-bili-comments

Web app displaying translated Bilibili video comments for Ludwig's "Tip to Tip" motorcycle series across China. Translates Chinese audience reactions into English with cultural annotations so international viewers can follow along.

**Stack:** Next.js 16 (App Router), React 19, Tailwind CSS 4, Radix UI, Bun, Drizzle ORM, Supabase (Postgres), Playwright, Claude API, oxlint, oxfmt
**Deploy:** Vercel, comment JSON on Cloudflare R2, images on R2, metadata in Supabase

## Critical Constraints

**Annotation marker/array parity:** Comments use `[~N:translated phrase]` markers in content that reference an `annotations[]` array on the comment object. Every marker MUST have a matching annotation entry (by index), and every annotation MUST have a marker in the content. Mismatches break the tooltip rendering in `AnnotatedText.tsx`. Validation logic lives in `lib/annotations.ts`. The translation prompt rules live in `tools/prompt-template.md`.

**Emote preservation:** Emote codes like `[笑哭]` must be preserved verbatim in translations. They are rendered as inline images via `emoteMap` lookup in `RichText.tsx`.

**Data architecture — Supabase metadata + R2 blobs:** Comment JSON blobs live in R2 (cheap storage). Supabase stores metadata only: R2 keys, comment counts, timestamps, categories, videos. `lib/comments.ts` queries Supabase for R2 keys, then fetches the JSON blob from R2. Overrides (approved edits) are stored in Supabase and merged at read time.

**Provider-agnostic schema:** The `videos` table uses a generic `id` + `provider` enum (`bilibili | youtube | other`) instead of Bilibili-specific fields. This supports future non-Bilibili sources.

**Server-only modules:** `lib/comments.ts` and `lib/config.ts` query Supabase — never import them from client components. All data loading happens server-side.

**URL rewriting:** `lib/comments.ts` rewrites Bilibili CDN URLs (`i0-i2.hdslb.com`) to `NEXT_PUBLIC_ASSETS_URL` at load time. For `new_dyn/` picture URLs, translated variants use `.en.jpg` suffix. Don't hardcode Bilibili CDN URLs in components.

**Translation resume:** `tools/translate.ts` tracks `translatedIds` in file metadata to skip already-translated comments on re-run. Don't break this field.

**Drizzle migrations:** Never manually edit files in `drizzle/` (migrations, journal, snapshots). Always use `bunx drizzle-kit generate` then `bunx drizzle-kit migrate`. Schema source of truth is `lib/db/schema.ts`.

**UI primitives — Radix UI:** Use `radix-ui` for dialogs, popovers, tooltips, and other interactive primitives. Import as `import { Dialog, Popover, Tooltip } from "radix-ui"` and use compound components (`Dialog.Root`, `Dialog.Content`, etc.). Each component manages its own open/close state locally — no global overlay store. Radix handles scroll lock, portals, focus trapping, and accessibility.

## Architecture & Data Flow

### Comment Pipeline

```
fetch-comments (Playwright + Bilibili API)
  -> comments/{categoryId}/{videoId}.json (local)
  -> translate (Claude Code CLI, batched)
  -> comments/{categoryId}/{videoId}.en.json (local)
  -> download-assets (Bilibili CDN images to .assets/)
  -> upload-assets:
     Phase 1: images to R2
     Phase 2: comment JSON to R2 → upsert comment_data + insert fetch_snapshots in Supabase
     Phase 3: danmaku XML to R2
```

### Server-Side Data Loading

```
Request hits Vercel
  -> lib/config.ts queries Supabase for categories + videos
  -> lib/comments.ts queries Supabase for R2 keys
  -> fetches JSON blobs from R2
  -> queries overrides table, merges approved edits
  -> rewriteCommentUrls() rewrites Bilibili CDN URLs
  -> Page renders with full data (no client-side fetching)
```

### Edit Flow

```
User submits edit → POST /api/edits (validates annotations)
  -> edits table (status: pending)
Admin approves → POST /api/admin/edits/[id]/approve
  -> overrides table (upsert)
```

### Danmaku Pipeline

```
Bilibili danmaku XML
  -> translate-danmaku (Claude)
  -> merge-danmaku (per-category merge)
  -> danmaku/{categoryId}.xml + danmaku/{categoryId}.en.xml
  -> danmaku/manifest.json (on R2)
```

### Database Schema

- `categories` — id, title, description, maxAge, sortOrder
- `videos` — id (provider-specific), provider enum, categoryId, title, url, sortOrder
- `comment_data` — videoId, lang, r2Key, commentCount, fetchedAt, translatedAt (current state)
- `fetch_snapshots` — videoId, lang, r2Key, commentCount, viewCount, likeCount, danmakuCount, fetchedAt (time-series)
- `edits` — videoId, commentId, content, annotations, status (pending/approved/rejected)
- `overrides` — videoId, commentId, content, annotations, editId (approved edits applied at read time)

### Directory Layout

- `app/` — Next.js App Router pages + API routes
- `app/login/` — Supabase auth login page
- `app/auth/callback/` — OAuth/email verification callback
- `app/admin/edits/` — Admin edit review dashboard
- `app/api/edits/` — Edit submission endpoint
- `app/api/admin/edits/` — Admin approve/reject endpoints
- `components/` — React components organized by domain. Client components use `"use client"`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [capybara-baron/t2t-bilibili-comments](https://github.com/capybara-baron/t2t-bilibili-comments) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
