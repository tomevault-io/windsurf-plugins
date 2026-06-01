---
trigger: always_on
description: **Community Archive** is a Twitter data preservation project that allows users to upload their Twitter archive exports to create a searchable public database. The goal is to preserve cultural/historical data from Twitter communities while providing open APIs for building apps on top of this data.
---

# Community Archive - Agent Knowledge File

## Project Overview

**Community Archive** is a Twitter data preservation project that allows users to upload their Twitter archive exports to create a searchable public database. The goal is to preserve cultural/historical data from Twitter communities while providing open APIs for building apps on top of this data.

**Live at**: https://www.community-archive.org/

### Core Technologies
- **Frontend**: Next.js 14 (App Router), React 18, TailwindCSS, shadcn/ui
- **Backend**: Supabase (PostgreSQL + Auth + Storage + Edge Functions)
- **Data Processing**: Server-side archive processing via Docker worker
- **State Management**: TanStack Query (React Query)
- **Testing**: Jest with Testing Library

### Key Data Flow
1. User authenticates via Twitter OAuth through Supabase Auth
2. User uploads their Twitter archive zip file
3. Archive is parsed client-side, stored in Supabase Storage as JSON
4. An `archive_upload` record is created with phase `ready_for_commit`
5. A Docker worker (`services/process_archive/`) picks up pending uploads
6. Worker processes the archive JSON and inserts data into PostgreSQL tables
7. Upload phase transitions through: `uploading` → `ready_for_commit` → `committing` → `completed`

### Database Schema (key tables)
- `all_account` - Twitter accounts that have uploaded archives
- `all_profile` - Profile data (bio, avatar, header, location)
- `tweets` - Main tweet table with full-text search (tsvector)
- `mentioned_users` / `user_mentions` - User mention relationships
- `tweet_media` / `tweet_urls` - Tweet attachments
- `liked_tweets` / `likes` - User likes
- `followers` / `following` - Social graph
- `quote_tweets` / `retweets` - Tweet relationships
- `optin` - Opt-in status for tweet streaming feature
- `global_activity_summary` - Materialized view for stats

### Key Directories
- `src/app/` - Next.js pages and API routes
- `src/lib/` - Business logic, utilities, queries
- `src/components/` - React components (UI primitives in `ui/`)
- `services/process_archive/` - Docker-based archive processor
- `sql/` - SQL function definitions (organized by feature)
- `supabase/` - Supabase config, migrations, schemas
- `scripts/` - CLI utilities and one-off scripts

### Hetzner inventory & worker placement (2026-05-23 audit)

| Host | RAM (free) | vCPU | Disk free | Resident workload | Suitable for |
| --- | --- | --- | --- | --- | --- |
| `ca-autorefresh` (95.217.12.23) | 3.7 GB (3.0 GB free) | 2 | 30 GB | `dashboard-app` (small Node) + `caddy` | **default home for admin/maintenance workers** — admin-delete-worker lives here |
| `prod-vector-store` (65.109.10.177) | 15 GB (3.0 GB free) | 4 | 58 GB | Qdrant (11.3 GB / 74% of cap), Coolify-managed app, Cloudflared tunnel | search/embeddings only — leave alone |
| `hetzner` (65.108.63.153) | unknown | ? | ? | unknown (key in ~/.ssh/config rotated; can't reach) | re-verify before assuming it exists |

**Don't put new workers on `prod-vector-store`.** Qdrant is at 74% of
its hard memory limit and grows with the corpus. Anything else
competing for the last 3 GB risks an OOM-kill that takes down vector
search.

**Long-term TODO: move the vector store off a continuously-running
Hetzner box onto on-demand compute (Modal or similar).** Per the
2026-05-23 conversation: we burst-use Qdrant every few months for
analytics/research, not continuously. Paying for a 15 GB box 24/7 is
wasteful. Modal's pay-per-use container model fits this pattern
better. Open question: does Qdrant's storage model survive container
sleep/wake? If not, persist the index to S3/R2 and reload on cold
start.

## Refactor Audit

### 🔴 Critical Issues

#### 1. Type Safety Erosion
```typescript
// services/process_archive/process_archive_upload.ts
type Sql = any  // Defeats TypeScript's entire purpose
```
The archive processor—the most critical part of the system—has no type safety for its database operations.

#### 2. Duplicated Supabase Config
`getSupabaseConfig()` is copy-pasted between:
- `src/utils/supabase.ts`
- `services/process_archive/process_archive_upload.ts`

Any environment variable changes require updating multiple files.

#### 3. Migration Drift
The `supabase/migrations-pending-review/` folder contains unapplied migrations dating back to March 2025. Schema changes are split across:
- `sql/tables/` (documentation)
- `supabase/schemas/` (declarative)
- `supabase/migrations/` (applied)
- `database.types.ts` (generated)

No single source of truth for schema.

#### 4. Dead/Disabled Code
```typescript
// process_archive_upload.ts:232
if (false && CONFIG.USE_COPY) { // COPY optimization disabled forever
```
The COPY optimization is permanently disabled but the code remains, adding 100+ lines of dead weight.

### 🟠 Architectural Issues

#### 5. Inconsistent Script Languages
The `scripts/` folder mixes `.js`, `.ts`, `.mts` files without clear reasoning:
- `check-tables.js` (JS)
- `import_from_files_to_db.ts` (TS)
- `download_supabase_storage.mts` (ESM TS)

#### 6. fp-ts Underutilization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheExGenesis/community-archive](https://github.com/TheExGenesis/community-archive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
