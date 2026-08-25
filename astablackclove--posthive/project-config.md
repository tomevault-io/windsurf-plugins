---
trigger: always_on
description: Social media scheduling SaaS. Schedule posts to Bluesky, Threads, Instagram, LinkedIn, Mastodon, YouTube, Facebook Pages, Pinterest, Telegram, Nostr, X (Twitter), Discord, Tumblr, and Lemmy from a single UI. Self-hostable, open-source (AGPL-3.0).
---

# Posthive — CLAUDE.md

Social media scheduling SaaS. Schedule posts to Bluesky, Threads, Instagram, LinkedIn, Mastodon, YouTube, Facebook Pages, Pinterest, Telegram, Nostr, X (Twitter), Discord, Tumblr, and Lemmy from a single UI. Self-hostable, open-source (AGPL-3.0).

---

## Monorepo Structure

```
social-scheduler/
├── apps/
│   ├── api/          Fastify v4 backend (Node.js, TypeScript, ESM)
│   ├── web/          Next.js 16 frontend (App Router, TypeScript)
│   └── mcp/          MCP stdio server (Claude Code / Cursor / local agents)
├── .claude/
│   └── TASKS.md      Feature backlog and completed work
└── CLAUDE.md         This file
```

---

## Running the Project

```bash
# Both apps in parallel
pnpm dev

# Individual
pnpm dev:api      # API on http://localhost:3001
pnpm dev:web      # Web on http://localhost:3000

# Database
cd apps/api
pnpm db:studio    # Prisma Studio at http://localhost:5555
pnpm db:migrate   # Run pending migrations
```

---

## API — `apps/api`

**Stack:** Fastify v4 · Prisma 5 · SQLite (dev) / Postgres (prod) · BullMQ · Redis (Upstash) · TypeScript ESM

**Entry:** `src/index.ts` — registers CORS, routes, starts worker

### Routes
| File | Prefix | Purpose |
|------|--------|---------|
| `routes/auth.ts` | `/auth/*` | JWT login/register + Threads, Instagram, LinkedIn, YouTube, Facebook, Pinterest, Mastodon, Twitter, Discord, Tumblr OAuth + Lemmy credential connect |
| `routes/accounts.ts` | `/accounts` | List/disconnect social accounts |
| `routes/jobs.ts` | `/jobs` | CRUD + reschedule + delete scheduled posts |
| `routes/upload.ts` | `/upload` | Image/video upload → local disk or Supabase Storage |
| `routes/billing.ts` | `/billing` | Dodo Payments checkout + webhook |
| `routes/user.ts` | `/user` | Profile info + `/auth/refresh` (rate-limited 20/15 min) |
| `routes/apiKeys.ts` | `/user/api-keys` | Create / list / revoke API keys (Pro/Team) |
| `routes/publicApi.ts` | `/api/v1` | Public REST API — accounts, posts, upload, templates |
| `routes/mcp.ts` | `/mcp` | MCP Streamable HTTP server (Bearer + key-in-URL) |
| `routes/oauth.ts` | `/oauth`, `/.well-known` | OAuth 2.0 + PKCE server for Claude.ai MCP connector |
| `routes/templates.ts` | `/templates` | Post template CRUD |

### Platform Adapters — `src/adapters/`
Each adapter implements `PlatformAdapter` from `types.ts`:
- `bluesky.ts` — AT Protocol (app password auth)
- `threads.ts` — Meta Threads API (OAuth 2.0, 60-day tokens)
- `instagram.ts` — Instagram Business API (OAuth 2.0, image/carousel/reel/story publishing)
- `linkedin.ts` — LinkedIn UGC API (OAuth 2.0, text + image posts)
- `mastodon.ts` — Mastodon API (OAuth 2.0, any instance; SSRF protection on instanceUrl)
- `youtube.ts` — YouTube Data API v3 (Google OAuth 2.0, resumable video upload; type = short/video)
- `facebook.ts` — Facebook Graph API v21.0 (OAuth 2.0, page access tokens, text/photo/video/carousel)
- `pinterest.ts` — Pinterest API v5 (OAuth 2.0, Pins with image required; sandbox mode supported)
- `telegram.ts` — Telegram Bot API (bot token + channel username, no OAuth)
- `twitter.ts` — X/Twitter API v2 (OAuth 1.0a HMAC-SHA1, up to 4 images; Pro/Team only)
- `nostr.ts` — Nostr protocol (keypair auth, Kind 1 notes, NIP-92 image tags, no OAuth)
- `discord.ts` — Discord webhook API (OAuth 2.0, webhook auto-created per channel)
- `tumblr.ts` — Tumblr API v2 (OAuth 1.0a HMAC-SHA1, NPF text + image posts; tokens never expire)
- `lemmy.ts` — Lemmy API v3 (username/password → JWT per post; community posts; any instance; no OAuth)

**Register adapters in `src/adapters/index.ts`** — add to the array to enable.

### Key Libraries
- **`lib/encryption.ts`** — AES-256-GCM encrypt/decrypt for credentials. Key from `ENCRYPTION_KEY` env (64-char hex).
- **`lib/storage.ts`** — `StorageAdapter` interface. `LocalDiskStorage` (dev) or `SupabaseStorage` (prod).
- **`lib/queue.ts`** — BullMQ queue setup. `schedulePostJob(id, date)` enqueues a job.
- **`lib/worker.ts`** — BullMQ worker. Processes jobs: refresh token → create post → create comment → update status.
- **`lib/auth/`** — `localAuth.ts` (JWT, bcrypt) or `supabaseAuth.ts`. Switch via `AUTH_PROVIDER` env.
- **`lib/plans.ts`** — plan limits (account count, etc.) per billing tier.

### Job State Machine (per PostJobTarget)
```
pending → running → post_done → comment_done (= done on PostJob)
                 ↘ post_failed
                              ↘ comment_failed
```

---

## MCP — `apps/mcp`

**Stack:** TypeScript ESM · `@modelcontextprotocol/sdk` stdio transport

Standalone stdio MCP server for local agents (Claude Code, Cursor). Calls the Posthive REST API using env vars. Run with:

```bash
cd apps/mcp
POSTHIVE_API_URL=https://your-api POSTHIVE_API_KEY=ph_xxx node dist/index.js
```

All 10 tools mirror the HTTP MCP server: `list_accounts`, `create_post`, `get_post`, `list_scheduled_posts`, `approve_draft`, `update_post`, `duplicate_post`, `delete_post`, `list_templates`, `create_from_template`.

---

## Web — `apps/web`

**Stack:** Next.js 16 App Router · React 18 · Tailwind CSS · TypeScript

### Pages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AstaBlackClove/posthive](https://github.com/AstaBlackClove/posthive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
