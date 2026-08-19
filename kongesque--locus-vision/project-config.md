---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
- `pnpm dev` - Start fullstack dev server (frontend + backend concurrently)
- `pnpm dev:frontend` - Start frontend only (Vite dev server on port 5173)
- `pnpm dev:backend` - Start backend only (FastAPI/uvicorn on port 8000 with reload)
- `pnpm build` - Production build (outputs to `dist/`)
- `pnpm preview` - Preview production build

**Docker alternative:** `docker compose up --build` — runs app on port 3000, API on port 8000.

**Backend venv:** `source backend/.venv/bin/activate` — required before all `cd backend && ...` commands.

**Python version:** 3.11 or 3.12 only (TFLite Runtime and ML packages have strict version requirements).

### Testing
- `pnpm test` - Run all frontend unit tests (Vitest with --run)
- `pnpm test:unit` - Run Vitest in watch mode
- `pnpm test -- src/path/to/file.test.ts` - Run a single frontend test file
- `cd backend && pytest` - Run all backend tests
- `cd backend && pytest tests/test_specific.py` - Run a single backend test file
- `cd backend && pytest tests/test_specific.py::test_function -v` - Run a single backend test

### Code Quality
- `pnpm lint` - ESLint + Prettier check
- `pnpm format` - Format all files with Prettier
- `pnpm check` - Type-check Svelte files with svelte-check
- `pnpm check:watch` - Type-check in watch mode

### Model & Benchmarking
- `python backend/scripts/export_model.py yolo11n --int8` - Export quantized model to `data/models/` (requires `ultralytics`)
- `python backend/scripts/benchmark_inference.py` - Benchmark ONNX inference on current hardware

## Code Style

Prettier enforces: **tabs** for indentation, **single quotes**, **no trailing commas**, **100 char printWidth**. Svelte files use the `svelte` parser. Tailwind class sorting is automatic via `prettier-plugin-tailwindcss`.

ESLint uses flat config format (ESLint 9). `no-undef` is off (TypeScript handles it).

## High-Level Architecture

LocusVision is a self-hosted video analytics platform targeting **Raspberry Pi 5 (8GB)**. All processing is local — no cloud dependencies.

### Frontend-Backend Communication

There is **no Vite proxy** — the frontend calls the backend directly. Both servers run independently on different ports (5173 for frontend, 8000 for backend). CORS is configured in `backend/config.py` for these origins.

`src/lib/api.ts` exports `API_URL` which resolves dynamically to `${window.location.hostname}:8000` on the client (so Docker deployments on non-localhost hosts work) and falls back to `http://127.0.0.1:8000` on the server. Always import `API_URL` from `$lib/api` rather than hardcoding the backend URL.

API calls from the frontend use native `fetch` (no axios or wrapper library). Server-side calls in `hooks.server.ts` and `+page.ts` load functions hit the backend directly. Client-side calls in components use the same pattern.

Interactive API docs are at `http://localhost:8000/api/docs`.

### Frontend (SvelteKit 5 + TypeScript)

**Authentication Flow**
- Authentication is handled in `src/hooks.server.ts` via JWT access/refresh tokens stored in HttpOnly cookies
- All routes except `/login`, `/signup`, `/get-started`, `/logout` require authentication
- The hook validates tokens against the FastAPI backend (`/api/auth/me`) and auto-refreshes expired access tokens
- User data is attached to `event.locals.user` and available in all route loaders
- User type defined in `src/app.d.ts`: `{ id: number; email: string; name: string; role: 'admin' | 'viewer' } | null`
- First-time setup: navigate to `/get-started` to create the initial admin account

**Route Structure**
- `src/routes/(app)/` - Authenticated application routes (livestream, video-analytics, analytics, settings, system)
  - `livestream/` - Camera grid view
  - `livestream/[taskId]/` - Single-camera fullscreen view
  - `video-analytics/` - Video job list and upload
  - `video-analytics/[taskId]/` - Video job detail and results
  - `create/[taskId]/` - Zone/line drawing canvas for configuring analytics on a video or stream before processing. Uses `$lib/stores/video.svelte.ts` (global store) to carry the video URL/type across the navigation from the upload step.
  - `analytics/` - DuckDB-backed event analytics dashboard
  - `settings/` - User profile, appearance, models, admin
  - `system/` - System health and storage
- `src/routes/(auth)/` - Public authentication routes (login, signup, get-started)
- `src/routes/+layout.svelte` - Root layout with global styles

**Component Architecture**
- UI components use shadcn-svelte (installed in `src/lib/components/ui/`) with bits-ui primitives
- Add new shadcn components with: `npx shadcn-svelte@latest add <component>`
- Custom components are organized by feature: `livestream/`, `video-analytics/`, `create/`
- Svelte 5 runes (`$state`, `$derived`, `$props`) are used throughout — avoid legacy `$:` reactive statements
- Styling uses Tailwind CSS 4 with CSS variables for theming (defined in `src/routes/layout.css`)
- Component aliases: `$lib/components`, `$lib/components/ui`, `$lib/hooks`

**State Management**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kongesque/locus-vision](https://github.com/kongesque/locus-vision) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
