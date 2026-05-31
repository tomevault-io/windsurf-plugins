---
trigger: always_on
description: Framedeck is an AI-powered video editor. Users edit videos with natural language prompts to remove silences, add b-roll, generate subtitles, create motion designs, analyze footage, prepare voiceovers, and render finished videos.
---

# Framedeck - Agent Guide

Framedeck is an AI-powered video editor. Users edit videos with natural language prompts to remove silences, add b-roll, generate subtitles, create motion designs, analyze footage, prepare voiceovers, and render finished videos.

This file is the root guide for coding agents. Keep changes small, preserve the existing architecture, and prefer local project patterns over new abstractions.

## Project Overview

Framedeck is a PNPM/Turborepo monorepo with a shared `ts-rest` contract.

```text
apps/
├── frontend/          # Next.js 16, React 19, Tailwind CSS v4, shadcn, Zustand, Remotion
├── server/            # NestJS 11, AI SDK, S3, Socket.IO, ElevenLabs Scribe v2, TwelveLabs
└── media-processor/   # Rust/Axum internal service for FFmpeg audio extraction
packages/
└── api-types/         # Shared ts-rest contracts, Zod schemas, realtime constants
```

The API contract in `packages/api-types` is the source of truth for frontend hooks and server handlers. Do not hardcode backend URLs in frontend code; use the generated `api` client and env-driven base URLs.

## Required Setup

Prerequisites:

- Node.js LTS and PNPM `10.22.0`
- Rust toolchain and Cargo
- FFmpeg for media extraction and local media workflows
- App-specific `.env` files copied from each `.env.example`
- Provider credentials as needed: AWS S3/Remotion, AI Gateway or OpenAI-compatible API, ElevenLabs, TwelveLabs, Deepgram

Install dependencies from the repo root:

```bash
pnpm install
```

The root does not own a single combined env file. Configure env files per app:

- `apps/frontend/.env`
- `apps/server/.env`
- `apps/media-processor/.env`

## Development Commands

Run the full stack:

```bash
pnpm dev
```

Run without portless URL injection:

```bash
pnpm dev:direct
```

Run individual apps:

```bash
pnpm --filter frontend dev
pnpm --filter server dev
pnpm --filter media-processor dev
pnpm --filter api-types dev
```

Common checks:

```bash
pnpm typecheck
pnpm test
pnpm build
pnpm --filter frontend lint
pnpm --filter server lint
pnpm --filter media-processor lint
```

Package-specific commands:

```bash
pnpm --filter api-types build
pnpm --filter frontend exec tsc
pnpm --filter server exec tsc
pnpm --filter server test
pnpm --filter server test:e2e
pnpm --filter server test:cov
pnpm --filter media-processor test
```

Portless dev URLs:

- Frontend: `http://ai-video-editor.localhost:1355`
- Backend: `http://api-ai-video-editor.localhost:1355`
- Media processor: `http://media-ai-video-editor.localhost:1355`

Use `pnpm portless:list` to inspect active routes.

## Architecture

### ts-rest API Workflow

When adding or changing an API route:

1. Update `packages/api-types/src/index.ts` with Zod schemas and the `apiContracts` route.
2. Build shared types with `pnpm --filter api-types build`.
3. On the server, bind the route with `@TsRestHandler(apiContracts.<router>.<endpoint>)`.
4. Keep business logic in services, not controllers.
5. On the frontend, consume through `api.<router>.<endpoint>.useQuery()` or `.useMutation()`.

Request and response shape changes must start in `api-types` so frontend and server compile against the same contract.

### Upload Pipeline

1. Client calls `POST /uploads/init` to create a multipart upload and receive presigned URLs.
2. Client calls `POST /uploads/:uploadId/sign-parts` to sign chunks.
3. Browser uploads directly to S3 through `directS3Upload()` with parallel multipart upload and Transfer Acceleration.
4. Client calls `POST /uploads/:uploadId/complete`.
5. Server starts ElevenLabs Scribe v2 transcription in the background.
6. Server starts TwelveLabs video analysis in the background for video assets.
7. Results arrive over WebSocket events such as `transcriptionComplete` and `upload:videoAnalysisComplete`.

Asset statuses may include `pending-upload`, `uploading`, `in-progress`, `transcribing`, `ready`, `uploaded`, and `error`. Check existing frontend/server usage before renaming or consolidating statuses.

### Video Analysis

- TwelveLabs analysis is triggered after `POST /uploads/:uploadId/complete` for video assets.
- It runs in parallel with ElevenLabs Scribe v2 transcription and must not block upload completion.
- The pipeline is presigned S3 GET URL, TwelveLabs indexing, parallel prompt analysis, then WebSocket `upload:videoAnalysisComplete`.
- Frontend stores analysis on `asset.summary` with fields such as `macroView`, `causalLogic`, `sequentialSummary`, `socket`, and `plug`.

### Agent-Driven Editor

- `ai-gateway.service.ts` streams text and reasoning through the AI SDK and calls tools from `ToolsService.getTools()`.
- `tools.service.ts` defines Zod-validated tools named with `editorToolNames`.
- Tool calls emit WebSocket start, delta, end, and result events.
- `selectTimelineItems` dispatches through `RealtimeService`.
- `removeSilences` waits for frontend results through `registerToolResult` using tool-call-id promises.
- `WebSocketProvider.tsx` routes realtime messages by `RealtimeMessageType`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kevinrss01/framedeck](https://github.com/kevinrss01/framedeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
