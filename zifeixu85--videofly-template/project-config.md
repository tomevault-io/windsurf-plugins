---
trigger: always_on
description: Handles video generation lifecycle.
---

# VideoFly - AI Video Generation Platform

## Project Overview

VideoFly is a SaaS platform for AI-powered video generation. It's built as a standalone Next.js application with AI video generation capabilities.

## Tech Stack

- **Framework**: Next.js 15 (App Router)
- **Runtime**: React 19
- **Language**: TypeScript
- **Database**: PostgreSQL with **Drizzle ORM**
- **Auth**: Better Auth + Google OAuth + Magic Link
- **Styling**: Tailwind CSS 4 + shadcn/ui (Radix UI)
- **Package Manager**: pnpm

## Project Structure

```
videofly/
├── src/
│   ├── app/                  # Next.js App Router pages
│   │   ├── api/              # API Routes
│   │   │   ├── v1/           # REST API v1
│   │   │   ├── auth/         # Better Auth endpoints
│   │   │   ├── trpc/         # Legacy tRPC
│   │   │   └── webhooks/     # Webhooks (Stripe, Creem)
│   │   └── [locale]/         # i18n pages (marketing, dashboard, tool)
│   ├── ai/                   # AI provider abstraction
│   │   ├── providers/        # evolink, kie providers
│   │   └── types.ts
│   ├── components/           # React components
│   ├── config/               # Configuration
│   │   ├── credits.ts        # Credit/Model pricing config
│   │   └── pricing-user.ts   # User-facing pricing config
│   ├── db/                   # Database
│   │   ├── schema.ts         # Drizzle schema
│   │   └── index.ts
│   ├── lib/                  # Utilities
│   │   ├── auth/             # Better Auth configuration
│   │   ├── storage.ts        # R2/S3 storage
│   │   └── ...
│   ├── payment/              # Payment integration
│   │   ├── index.ts          # Stripe client
│   │   ├── plans.ts          # Subscription plans
│   │   └── webhooks.ts       # Stripe webhooks
│   ├── services/             # Business services
│   │   ├── credit.ts         # Credit system (freeze/settle/release)
│   │   ├── video.ts          # Video generation lifecycle
│   │   └── billing.ts
│   ├── stores/               # Zustand state stores
│   ├── hooks/                # React hooks
│   ├── i18n/                 # Internationalization
│   └── middleware.ts
├── scripts/                  # Utility scripts
├── docs/                     # Documentation
└── public/                   # Static assets
```

## Core Modules

### 1. AI Provider Layer (`src/ai/`)

Unified abstraction for multiple AI video generation providers.

**Supported Providers:**
- **evolink** - Primary provider
  - Sora 2 (image-to-video, 10-15s, 16:9/9:16)
  - Wan 2.6 (image-to-video, 5-15s, multiple ratios)
  - Veo 3.1 (short clips, 4-8s)
  - Seedance 1.5 Pro (multiple qualities 480P-1080P)
- **kie** - Secondary provider

**Key Files:**
- `index.ts` - Provider factory
- `providers/evolink.ts` - Evolink implementation
- `providers/kie.ts` - Kie implementation
- `types.ts` - Interface definitions

**Usage:**
```typescript
import { getProvider } from "@/ai";
const provider = getProvider("evolink");
const task = await provider.createTask({ prompt, duration, aspectRatio });
```

### 2. Credit System (`src/services/credit.ts`)

FIFO-based credit management with freeze/settle/release pattern.

**Features:**
- Credit packages with expiration dates
- Freeze credits during video generation
- Settle on success, release on failure
- Transaction history tracking
- Expiring credits warning

**Key Methods:**
- `getBalance(userId)` - Get available/frozen/used credits
- `freeze({ userId, credits, videoUuid })` - Freeze credits for task
- `settle(videoUuid)` - Confirm credit consumption
- `release(videoUuid)` - Release frozen credits on failure
- `recharge({ userId, credits, orderNo, transType })` - Add credits from purchase

### 3. Video Service (`src/services/video.ts`)

Handles video generation lifecycle.

**Flow:**
1. `generate()` - Create task, freeze credits, call AI API
2. AI provider processes asynchronously
3. `handleCallback()` - Receive completion webhook
4. `tryCompleteGeneration()` - Download video, upload to R2, settle credits

**Key Methods:**
- `generate(params)` - Start video generation
- `handleCallback(provider, payload, videoUuid)` - Process AI callback
- `refreshStatus(uuid, userId)` - Poll status for frontend
- `listVideos(userId, options)` - Get user's videos

### 4. Storage (`src/lib/storage.ts`)

R2/S3-compatible storage for video files.

**Features:**
- Presigned upload URLs
- Download from AI provider and re-upload to R2
- Public URL generation

## Authentication (`src/lib/auth/`)

**Providers:**
- Google OAuth
- Magic Link (email)
- Creem payment integration

**Key Files:**
- `auth.ts` - Better Auth configuration with Creem plugin
- `index.ts` - Server-side auth helpers (`getCurrentUser`, `requireAuth`)

## Payment Channels

### Creem (Primary)
- `@creem_io/better-auth` plugin
- Handles subscriptions and one-time credit purchases
- Webhook automatically credits user account

### Stripe (Secondary/Backup)
- Full Stripe SDK integration
- Stripe Webhooks support
- Customer/subscription management

## API Routes

### REST API (`/api/v1/`)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/video/generate` | POST | Start video generation |
| `/api/v1/video/list` | GET | List user's videos |
| `/api/v1/video/[uuid]` | GET/DELETE | Get/delete video |
| `/api/v1/video/task` | GET | Get task status |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zifeixu85/videofly-template](https://github.com/zifeixu85/videofly-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
