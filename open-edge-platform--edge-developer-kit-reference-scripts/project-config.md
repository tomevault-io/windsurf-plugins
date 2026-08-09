---
trigger: always_on
description: This is **Demo Studio** — an AI demo platform with a Next.js frontend that orchestrates Python-based AI worker services (embedding, speech-to-text, text-to-speech, image generation, etc.). The frontend manages service lifecycle, health monitoring, and provides interactive demo UIs for each AI capability.
---


# Project Guidelines

## Overview

This is **Demo Studio** — an AI demo platform with a Next.js frontend that orchestrates Python-based AI worker services (embedding, speech-to-text, text-to-speech, image generation, etc.). The frontend manages service lifecycle, health monitoring, and provides interactive demo UIs for each AI capability.

---

## Part 1: Codebase Structure

### Monorepo Layout

- `frontend/` — Next.js 16 app (App Router) with Payload CMS (SQLite), React 19, TypeScript
- `workers/` — Standalone Python FastAPI services spawned as child processes by the frontend

### Frontend (`frontend/`)

- **App Router** with route groups: `(dashboard)/` for main UI, `(payload)/` for CMS admin, `api/` for route handlers
- **Payload CMS** (SQLite) manages service state (status, ports, models, health checks) as the source of truth
- **Engine system** (`src/engines/`) abstracts inference backends
- **Service registry** (`src/services/`) — convention-based auto-discovery. Each service lives in `src/services/<name>/` with `data.ts` + `demo.tsx`
- **Use case registry** (`src/samples/`) — same pattern, `data.ts` with `sample` export
- **Context providers** (`src/context/`): `ServiceStatusContext` (polls service state), `SettingsContext` (localStorage), `SystemInfoContext` (OS/device detection)
- **API proxying**: `next.config.ts` rewrites `/api/<service-name>/*` to worker `http://localhost:<port>/*`

### Workers (`workers/`)

- Each worker is a standalone **FastAPI** server started via **UV** package manager
- Frontend's `process-handler.ts` spawns workers as child processes with rotating log streams
- Workers expose REST endpoints (e.g., `/v1/stt/transcribe`, `/v1/embeddings`)
- Two execution modes: `"worker"` (standalone process) or `"multiserve"` (shared OpenVINO engine)

### Tech Stack

#### Frontend

- Next.js 16, React 19, TypeScript 5
- Tailwind CSS 4, shadcn/ui (New York style, lucide icons), Radix UI
- Payload CMS 3.x with SQLite
- Eslint linting/formatting
- React Compiler enabled (babel-plugin-react-compiler)
- TanStack React Query for data fetching

#### Workers

- Python 3.11+, FastAPI, Uvicorn
- UV as package manager (not pip)
- OpenVINO, PyTorch, HuggingFace ecosystem
- Per-worker `pyproject.toml` for dependencies

### Service Lifecycle

Status flow: `inactive` → `prepare` → `active` (or `error`), with `restart` as a transition state.

- Start/stop via `PATCH /api/services/[id]` with `{ action: "start" | "stop" | "restart" }`
- Payload hooks trigger process spawn/kill
- Health check service polls worker endpoints every 10 seconds with JSONata-based response validation
- `ServiceStatusContext` polls `/api/services` every 5 seconds for UI updates

### Key Files Reference

- `src/services/types.ts` — Core `Service`, `ServiceMeta`, `WorkerConfig` types
- `src/engines/types.ts` — `Engine`, `EngineBackend` types
- `src/lib/process-handler.ts` — Worker process lifecycle management
- `src/lib/healthcheck.ts` — Health polling with JSONata evaluation
- `src/lib/constants.ts` — Paths (`WORKER_DIR`, `MODELS_DIR`, `LOGS_DIR`, `UV_PATH`), allowed ports
- `src/payload/collections/Services.ts` — Payload CMS service schema and hooks
- `scripts/generate-registries.mjs` — Auto-discovery codegen script
- `src/services/_generated/` — Auto-generated registries (do not edit manually)
- `src/engines/_generated/` — Auto-generated engine registry (do not edit manually)
- `src/samples/_generated/` — Auto-generated sample registry (do not edit manually)

---

## Part 2: Development Guidelines

### Build & Development Commands

```bash
# Frontend
cd frontend
npm run dev          # Start dev server
npm run build        # Production build (stop dev server first — .next/lock conflict)
npm run codegen      # Regenerate service/sample/engine registries
npm run lint         # eslint check
npm run lint:fix     # eslint auto-fix
```

**Always run `npm run lint` before committing** to ensure code passes eslint checks. For a smooth development experience, install the recommended VS Code extensions when prompted (eslint, Tailwind CSS IntelliSense, Payload, etc.).

### TypeScript / React Conventions

- Use **Biome** rules: 2-space indentation, organized imports, no `console.*` (use `src/lib/logger.ts` instead)
- Path aliases: `@/*` maps to `src/*`, `@payload-config` maps to `src/payload.config.ts`
- Prefer Server Components by default (RSC enabled). Use `"use client"` only when needed for interactivity
- UI components go in `src/components/ui/` (shadcn primitives) or `src/components/dashboard/` (app-specific)
- Use `cn()` from `@/lib/utils` for conditional class merging (clsx + tailwind-merge)
- Service demo components receive `{ service: Service }` prop and use `DemoParameterSidebar` for controls
- Never import from `_generated/services.ts` in server config contexts — use `_generated/meta.ts` instead (it has no React dependencies)

### API Calls — TanStack Query

All API calls in the frontend **must** use TanStack React Query. Never fetch directly inside components.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-edge-platform/edge-developer-kit-reference-scripts](https://github.com/open-edge-platform/edge-developer-kit-reference-scripts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
