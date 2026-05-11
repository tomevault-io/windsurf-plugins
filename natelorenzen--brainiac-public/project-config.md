---
trigger: always_on
description: > Shared patterns: `../../LESSONS.md` | Stack reference: `../../STACK.md`
---

# Brainiac — Project Context for Claude Code

> Shared patterns: `../../LESSONS.md` | Stack reference: `../../STACK.md`

---

## What This Is

A free, non-commercial YouTube thumbnail brain activation analyzer.
Users enter a YouTube channel handle. The app pulls the 25 most recent videos via the
YouTube Data API, runs each thumbnail through BERG (fmri-nsd-fwrf, CC-BY-NC-4.0) on
Modal CPU workers, then correlates each brain region's activation score against the
video's actual view count — showing which visual signals statistically track with
performance on that specific channel.

The Video Analyzer tab uses Meta FAIR TRIBE v2 (CC-BY-NC-4.0) on Modal GPU workers for
uploaded video files.

**Two Modal workers:**
- `BrainiacThumbnailInference` — BERG, CPU, YouTube thumbnails. Weights in `brainiac-berg-weights` volume.
- `BrainiacInference` — TRIBE v2, L4 GPU, uploaded videos. Weights in `brainiac-tribe-weights` volume.

**No revenue is generated. No performance claims are made. CC-BY-NC-4.0 on both models.**

**Live URL:** https://your-deployment.vercel.app
**GitHub:** https://github.com/your-org/brainiac-public
**Operator:** [YOUR COMPANY NAME]

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 16.2.1 (App Router) |
| Database & Auth | Supabase (Postgres + RLS + auth.users) |
| Storage | Supabase Storage (bucket: `heatmaps` only — thumbnails now fetched directly from YouTube by Modal) |
| Thumbnail Inference | Modal CPU — BERG fmri-nsd-fwrf (`BrainiacThumbnailInference`) |
| Video Inference | Modal L4 GPU — TRIBE v2 (`BrainiacInference`) |
| Hosting | Vercel |
| Styling | Tailwind CSS v4 |
| Charts | Recharts |
| Language | TypeScript (Next.js) + Python (Modal worker) |

**No Stripe. No Redis. No separate backend. No Cloudflare R2.**

**Note:** Next.js 16 has breaking changes. `middleware.ts` is renamed to `proxy.ts`;
export is `proxy`, not `middleware`.

---

## Environment Variables

All required vars are in `.env.local`. See `.env.example` for full list.

```bash
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=          # server-only, never expose client-side

MODAL_THUMBNAIL_URL=https://your-org--brainiac-thumbnail-inference.modal.run
MODAL_INFERENCE_URL=https://your-org--brainiac-inference.modal.run

ENCRYPTION_KEY=                     # 64-char hex (openssl rand -hex 32)

META_APP_ID=
META_APP_SECRET=
META_REDIRECT_URI=

YOUTUBE_DATA_API_KEY=               # Required — channel resolution + video list + view counts

NEXT_PUBLIC_APP_URL=
MONTHLY_BUDGET_CAP_USD=300.0
COST_PER_ANALYSIS_USD=0.0002        # BERG CPU inference ~$0.0002/thumbnail (was $0.03 on T4)
CURRENT_LEGAL_VERSION=1.0.0
```

### Modal Secrets

**`brainiac-supabase`** (both workers)
- `SUPABASE_SERVICE_ROLE_KEY`

**`brainiac-hf`** (BrainiacInference / video worker only)
- `HF_TOKEN` — HuggingFace read token (requires LLaMA 3.2-3B access approval from Meta)
- `MOCK_MODE` — set to `true` to use image-statistics fallback, `false` for real TRIBE v2

BERG thumbnail worker reads `MOCK_MODE` from the environment but does **not** require `HF_TOKEN`.

---

## Database Schema

### `profiles` (extends auth.users)
```sql
id, email, daily_count, monthly_count, daily_reset_at, monthly_reset_at,
account_status ('active'|'suspended'|'deleted'), deletion_requested_at, deletion_scheduled_at
```

### `user_consents`
```sql
user_id, consent_type ('terms_of_service'|'privacy_policy'|'data_aggregation'|'ad_account_connection'),
consented_at, ip_address, user_agent, legal_version
```

### `analyses`
```sql
user_id, type ('thumbnail'|'channel_batch'|'ad_creative'),
status ('queued'|'processing'|'complete'|'failed'),
input_storage_key, heatmap_storage_key, heatmap_url,
roi_data (JSONB), mean_top_roi_score, source, error_message
```

### `monthly_budget`
```sql
month (UNIQUE), analyses_run, estimated_cost_usd, budget_cap_usd (300.0), is_exhausted
```

### `connected_accounts`, `ad_creatives`, `creative_performance`
OAuth tokens encrypted at rest with AES-256-GCM. See `src/lib/encryption.ts`.

### `aggregate_signals`
Anonymized only — no user_id, no creative_id. Written after analyses with performance data.

### RPC functions (003_rpc_functions.sql)
- `increment_usage_counts(uid, n)` — atomic daily/monthly counter increment
- `increment_budget(p_month, p_cost, p_count)` — atomic budget increment

---

## Route Map

### Public pages (no auth)
```
/                   Landing page
/auth/login
/auth/signup
/auth/reset-password
/auth/update-password
/legal/terms
/legal/privacy
```

### Authenticated app (gated by proxy.ts)
```
/dashboard          Main analysis UI (upload + YouTube channel)
/account            Settings (data export, deletion, connected accounts)
```

### API routes
```
POST /api/analyze/thumbnail           Upload image → queue inference
GET  /api/analyze/[id]               Poll analysis status
POST /api/analyze/channel            YouTube channel batch analysis
GET  /api/users/me/usage             Daily/monthly cap status
GET  /api/users/me/consent           Check consent status
POST /api/users/me/consent           Record consents
GET  /api/users/me/data-export       Full JSON export (GDPR/CCPA)
DELETE /api/users/me                 Schedule 30-day account deletion

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [natelorenzen/brainiac-public](https://github.com/natelorenzen/brainiac-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
