---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ThreatPad is a collaborative, real-time note-taking app for cybersecurity / CTI teams. It features a Tiptap-based Markdown editor with Yjs collaboration, IOC auto-extraction, STIX 2.1 export, structured CTI templates, RBAC, and audit logging.

## Monorepo Structure

pnpm workspaces + Turborepo monorepo with four workspace packages:

- **`packages/shared`** (`@threatpad/shared`) — Domain types, Zod validators, constants (IOC regex patterns, system templates, role definitions), and utilities (IOC extraction, defang/refang)
- **`packages/db`** (`@threatpad/db`) — Drizzle ORM schema definitions, migrations, and seed script for PostgreSQL
- **`apps/web`** (`@threatpad/web`) — Next.js 15 frontend (App Router, React 19, TypeScript)
- **`apps/server`** (`@threatpad/server`) — Fastify 5 backend with REST API and Yjs WebSocket server

The shared package is consumed via `workspace:*` protocol and must be transpiled by Next.js (`transpilePackages` in next.config.ts).

## Commands

```bash
# Install dependencies
pnpm install

# Development (all workspaces)
pnpm dev

# Development (frontend only)
pnpm --filter @threatpad/web dev    # http://localhost:3000

# Development (backend only)
pnpm --filter @threatpad/server dev # http://localhost:3002

# Start local Postgres + Redis
docker compose up -d

# Database operations
pnpm --filter @threatpad/db push    # Push schema to DB
pnpm --filter @threatpad/db seed    # Seed demo data
pnpm --filter @threatpad/db studio  # Drizzle Studio GUI

# Production build
pnpm build

# Production Docker (all services)
docker compose -f docker-compose.prod.yml up -d

# Lint / type-check
pnpm lint
```

There are no tests yet.

## Architecture

### Frontend (`apps/web`)

**Routing:** Next.js App Router with two route groups:
- `(auth)` — public pages: `/login`, `/register`, `/forgot-password`, `/reset-password`, `/verify-email`
- `(app)` — authenticated layout with sidebar + header: `/dashboard`, `/workspace/[workspaceId]/**`
- `/oauth/callback` — handles OAuth redirect with accessToken param

**State management:**
- `Zustand` stores in `src/stores/` — `auth-store` (user session, JWT token) and `ui-store` (sidebar state, active workspace, command palette)
- All pages are wired to the real backend API via `src/lib/api-client.ts`

**Editor:** Tiptap 3 (ProseMirror-based) in `src/components/editor/`. Key config:
- `immediatelyRender: false` is required to avoid SSR hydration errors
- Extensions: StarterKit, CodeBlockLowlight (syntax highlighting via lowlight), Tables, TaskList, Highlight, Link, Image, Placeholder, ExcalidrawBlock
- Edit/Preview toggle — Edit mode shows WYSIWYG editor with toolbar; Preview mode renders content as clean read-only HTML with Tailwind Typography (`prose prose-invert`)
- Content is stored as HTML in the `contentMd` field (not raw markdown)
- Debounced auto-save (1 second) on both content and title changes
- Image upload: paste, drag-drop, or toolbar button → uploads to server via `POST /api/workspaces/:id/uploads`, stores on disk, references by authenticated URL
- Collaboration-ready (Yjs + y-websocket installed but not yet connected)

**Drawing support:** Two modes of drawing via `@excalidraw/excalidraw`:
- **Full-page drawings** — Notes with `type: 'drawing'` open a full-screen Excalidraw canvas instead of the text editor. Created via "New Drawing" in sidebar, folder context menu, or command palette. Drawing data stored as JSON in `contentMd`.
- **Embedded drawing blocks** — Text notes can contain inline drawing blocks via the `ExcalidrawBlock` Tiptap extension. Insert via toolbar "Insert Drawing" button. Shows a preview card in the editor; click "Edit" opens a full-screen Excalidraw modal. Drawing data stored as a JSON attribute on the node, serialized as `<div data-type="excalidraw" data-content="...">`.
- Excalidraw MUST be dynamically imported with `next/dynamic` + `ssr: false` (uses browser APIs)
- Dark theme (`theme="dark"`) to match ThreatPad
- Drawing editor debounces saves at 2 seconds (vs 1 second for text)
- `pnpm.overrides` in root `package.json` pins `@tiptap/core@3.20.4` to prevent version conflicts

**UI components:** Radix UI primitives wrapped with Tailwind in `src/components/ui/` (shadcn/ui pattern). Use `cn()` from `src/lib/utils.ts` for class merging.

**Styling:** Tailwind CSS 4 with `@tailwindcss/typography` plugin and custom theme variables in `src/styles/globals.css`. Dark mode is the default (class `dark` on `<html>`). Primary color: `#6366f1` (indigo).

**API client:** `src/lib/api-client.ts` — typed fetch wrapper (get, post, patch, put, delete, upload) with automatic JWT refresh on 401. Reads token from Zustand auth store. Base URL from `NEXT_PUBLIC_API_URL`. Important: Content-Type header is only set when request has a body (DELETE requests with no body must not set it). The `upload()` method uses FormData for multipart file uploads (does not set Content-Type — browser handles the boundary).

**Cross-component communication:** Child pages dispatch `CustomEvent` on `window` to notify the layout to refresh data:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bhavikmalhotra/ThreatPad](https://github.com/bhavikmalhotra/ThreatPad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
