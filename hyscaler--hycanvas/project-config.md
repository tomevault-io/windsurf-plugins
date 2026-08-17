---
trigger: always_on
description: Guidance for working in the HyCanvas repository.
---

# CLAUDE.md

Guidance for working in the HyCanvas repository.

## What This Project Is

HyCanvas is a free, self-hostable, AI-native design platform, built to lead on performance, AI quality, collaboration, openness, and accessibility. Everything is free: no tiers, no paywalls, no watermarks. Web-only.

Current state: the core product is built and runs. A single-player editor, content systems (uploads, stock, templates), accounts and workspaces, document types (presentations, video, whiteboard, docs, sheets), export, brand kits, and a bring-your-own-key AI layer all ship today. The remaining and early-stage work is tracked in `docs/roadmap/`.

## Source of Truth (read these first)

1. `README.md` - how to run the project (dev and production), the repository layout, environment variables, and the build/deploy story.
2. `docs/roadmap/` - forward-looking specs for work that is not yet built (realtime collaboration, AI media, accessibility/i18n/enterprise/NFR). Read the relevant spec before building in those areas.

For anything already shipped, the code is the source of truth; match the patterns of the surrounding code.

## Tech Stack

- Frontend: Next.js (React, Pages Router) + TypeScript, Zustand (editor state), Tailwind (UI chrome only, never for canvas content). Statically exported (`output: "export"`) for production.
- Rendering: custom scene-graph engine (`@hc/engine`) on Canvas2D, framework-agnostic so it runs in browser, worker, and headless on the server. A WebGL/WebGPU accelerated path is on the roadmap.
- Backend: Go (`backend`) - one service owning REST under `/api/v1`, the `/realtime` WebSocket, the Go rendering engine for export, and SQL migrations. chi router, pgx for Postgres. There is no Node API in the runtime.
- Realtime: a WebSocket relay with presence and locks ships; the full Yjs CRDT / offline-first model is on the roadmap (`docs/roadmap/16-realtime-collaboration.md`).
- Data: Postgres (metadata), S3-compatible object storage for assets/exports, with a local-filesystem fallback when no S3 is configured.
- Jobs: long work (export, video render, bulk create) runs through an in-process job registry, polled via `GET /api/v1/jobs/:id`.
- AI: a provider-adapter layer supporting built-in models and bring-your-own keys/endpoints; keys are stored encrypted per workspace, never via env.
- Auth: cookie sessions (httpOnly access + refresh with rotation), OIDC SSO, MFA (TOTP).
- Packaging: a single self-contained Go binary with the frontend embedded (`go:embed`, built `-tags embed`); the binary self-loads `.env`. Self-host via docker-compose.

## Monorepo Layout

The frontend and shared packages are an npm-workspaces monorepo (orchestrated with concurrently + dotenv-cli against a single shared root `.env`); the backend is a standalone Go module.

- `frontend` - Next.js app (Pages Router); statically exported for production and embedded into the Go binary.
- `backend` - the Go backend (REST, `/realtime`, export engine, DB migrations). Serves the embedded frontend in the production bundle. Postgres only.
- `packages/schema` - open file-format types and migrations (`@hc/schema`), no runtime deps.
- `packages/engine` - rendering engine (`@hc/engine`), no React/UI dependency.
- `packages/sdk` - typed REST/WS client (`@hc/sdk`).
- `packages/config` - typed, validated env config (`@hc/config`).
- `packages/ui` - shared UI utilities/components (`@hc/ui`).
- other `packages/*` - framework-agnostic `@hc/*` libraries (text, color, geometry, export, media, stock, templates, authz, formula, sheets, timeline, whiteboard, docs, publishing, website, print, a11y, ...). The frontend imports them from their built `dist/`.
- `scripts/build-dist.js` - embeds the exported frontend into the Go binary (`go build -tags embed`) and writes the single `dist/hycanvas`.

Keep the rendering engine free of any React or UI dependency so it stays reusable across browser, worker, and server.

## Key Architectural Rules

- The open design file format (`@hc/schema`) is the contract. Any feature that adds a node type or property must extend the schema and provide a forward migration. Opening an older file must always succeed.
- The database stores design snapshots in the open file format; restore, branch, export, and the API all reuse that format.
- Long-running work goes through the job registry, never inline in a request handler.
- Per-workspace data isolation is enforced at the query layer.
- Degrade gracefully: WebGL/WebGPU unavailable falls back to Canvas2D; object storage is abstracted so self-hosters can use local files or MinIO.

## Zero Data Loss (non-negotiable)

Every instance is someone's production instance, and self-hosters upgrade by swapping a binary. A change must be deployable onto a live instance holding real designs without destroying, corrupting, or silently altering any of it.

Rules for every change:
- **Never break existing data.** Opening, rendering, and saving a design created by any earlier version must keep working. If a change cannot preserve existing data, it does not ship in that form.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyscaler/HyCanvas](https://github.com/hyscaler/HyCanvas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
