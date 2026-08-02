---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`chats.is` — a multi-provider AI chatbot (Next.js App Router + React 19) that does chat, image, video, and audio/speech generation. It's an admin-configurable platform: providers, models, prompts, plans, pricing, and quotas all live in the database and are managed through a `/console` admin UI rather than hardcoded. Package manager is **pnpm**.

## Tech stack

- **Framework**: Next.js 16 (App Router, RSC, Turbopack), React 19, TypeScript (strict).
- **AI**: Vercel AI SDK v6 (`ai`, `@ai-sdk/*`) across OpenAI / Azure / Google / Vertex / Anthropic / Bedrock / xAI / DeepSeek.
- **API layer**: tRPC v11 + TanStack React Query; `superjson` transformer.
- **Database**: Postgres via Neon serverless driver, Drizzle ORM + drizzle-kit migrations.
- **Auth**: NextAuth v5 (beta) with Drizzle adapter, JWT sessions; Resend for email-code login.
- **Storage**: Vercel Blob for generated/uploaded media.
- **UI**: Tailwind CSS v4 (config-less), shadcn/ui (new-york) over Radix primitives, lucide + `@lobehub/icons`, framer-motion, sonner (toasts), CodeMirror (artifact code editing), react-markdown + remark-gfm.
- **Forms/validation**: react-hook-form + zod (zod also validates env and message-part schemas).
- **Tooling**: ESLint (flat config) + Prettier (with import-sort & tailwind plugins), Vitest (node env), `@t3-oss/env-nextjs` for env validation.

## Commands

```bash
pnpm dev                 # dev server (Next 16 / Turbopack)
pnpm build               # production build
pnpm vercel-build        # drizzle migrate + build (deploy)
pnpm lint                # eslint
pnpm type-check          # tsc --noEmit  (run this to verify type changes)
pnpm test                # vitest run (all tests)
pnpm test:watch          # vitest watch
pnpm format:write        # prettier write
pnpm db:generate         # generate migration from schema.ts changes
pnpm db:migrate          # apply migrations
pnpm db:studio           # drizzle studio
```

Run a single test: `pnpm vitest run lib/pricing.test.ts` (or `-t "name"` to filter).

Tests are **Node-environment unit tests** (vitest.config.ts), currently all in `lib/**` (billing/quota/usage/pricing logic), no DOM — coverage is *configured* for `lib/**` and `server/**`, but no `server/` tests exist yet. `server-only` is stubbed and `@electric-sql/pglite` is available for DB-backed tests (see `billing.integration.test.ts`). There are no component/E2E tests.

## Architecture

### Request → response is split by modality
- **Chat is streaming.** `app/api/chat/route.ts` (~800 lines, the heart of the app) uses Vercel AI SDK v6: `streamText` wrapped in `createUIMessageStream` → `createUIMessageStreamResponse`. It also defines the `create_artifact` tool inline and streams artifact deltas as custom data parts.
- **Image / video / audio / speech are non-streaming.** `app/api/{images,video,audio,speech}/route.ts` all follow one shape: generate → upload bytes to **Vercel Blob** (`@vercel/blob` `put`) → build an `assistant` message whose `parts` contains a `file` part with the blob URL → persist → record usage → return JSON. The client (`lib/api.ts`) awaits the whole result.

Every generation route follows the same lifecycle: `auth()` → look up model/provider → **`preflightGate()`** (pricing exists + model allowed + quota not exceeded; returns 403/429) → generate → persist message(s) → **`recordXUsage()`**. Quota is checked *before* generation and consumed *after* (next request sees the new balance).

### Provider abstraction (`lib/provider.ts`)
All eight provider types (`openai`, `azure`, `google`, `vertex`, `anthropic`, `bedrock`, `xai`, `deepseek`) are unified behind `createProviderSDK(config)` → `getLanguageModel` / `getImageModel` / `getVideoModel` / `getSpeechModel`. **Config comes from the DB `providers` table, never env vars.** API keys are stored AES-encrypted (`lib/crypto.ts`) and `decrypt()`-ed at call time. `vertex` and `bedrock` pack their credentials as JSON in the `apiKey` field; their model IDs are remapped via `VertexAIModels` / `BedrockModels` in `lib/constant.ts`.

### Data model (`server/db/schema.ts`, Drizzle + Postgres via Neon serverless)
- DB driver is fixed to `@neondatabase/serverless` Pool + `drizzle-orm/neon-serverless` (`server/db/index.ts`). Schema changes → `pnpm db:generate` → migration in `server/db/migrations/`.
- **Messages**: `message.parts` is a JSONB column typed as `ChatMessage['parts']`. `ChatMessage = UIMessage<MessageMetadata, CustomUIDataTypes>` (extends the AI SDK `UIMessage`). Part schemas live in `types/content-part.ts` (text / reasoning / file / tool / source / data…). `message.parentId` is a self-FK enabling regenerate/branch threading.
- **Models & providers**: a `model` has a `capability` (chat/image/video/audio) plus two JSONB blobs — `uiOptions` (sizes/aspectRatios/resolutions/durations/voices used to drive the UI selectors) and `apiParams` (temperature/topP/…). Models belong to a `provider`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [copilot-is/copilot.is](https://github.com/copilot-is/copilot.is) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
