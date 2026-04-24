---
trigger: always_on
description: Purpose: concise, high-signal defaults for this Next.js 16 + TypeScript + Tailwind app. Keep scope tight, stay type-safe, avoid regressions.
---

# Atlantis AGENTS Guide

Purpose: concise, high-signal defaults for this Next.js 16 + TypeScript + Tailwind app. Keep scope tight, stay type-safe, avoid regressions.

## Quick Facts

- Framework: Next.js 16 App Router, TypeScript strict, moduleResolution bundler.
- Styling: Tailwind v4 + Shadcn UI primitives in `src/components/ui`.
- State: Zustand (`src/lib/store.ts`) for persisted settings.
- Diagrams: Mermaid editor; persisted to Prisma SQLite or JSON backup.
- Notes: see `src/app/notes`, server layouts fetch data and render client shells.
- Icons/Fonts/Themes: lucide-react, `next/font`, `next-themes` (light/dark toggle).
- Code map: `plan/code-map.md` is the required map-first index for file ownership and task routing.
- Editors: no Cursor rules (`.cursor/` missing); no Copilot instructions (`.github/copilot-instructions.md` missing). This file is authoritative.

## Mandatory Code Map Workflow

- Always read `plan/code-map.md` before searching or editing code.
- Use map-first navigation: start with the intent map and open those files before broad searching.
- Avoid blind repo-wide searches unless the code map does not cover the task.
- If you add/move/rename files, routes, folders, or exported functions, update `plan/code-map.md` in the same change.
- Treat stale code-map entries as bugs; fix them as part of the task.

## Commands (npm)

- Install deps: `npm install` (first step before any command).
- Dev server: `npm run dev` (runs `node scripts/bootstrap.js` then `next dev`, http://localhost:3000).
- Build (type-safety gate): `npm run build` (runs bootstrap then Next build).
- Start prod: `npm run start` (after build).
- Lint: `npm run lint` (ESLint via Next config).
- Prisma helpers: `npm run prisma:prepare`, `prisma:generate`, `prisma:push`, `prisma:migrate`, `migrate:json-to-db`.

## Testing Guidance

- No test runner configured. If tests are requested, add Vitest + @testing-library/react for Next 16.
- Suggested scripts once added: `"test": "vitest"`, `"test:watch": "vitest watch"`, `"test:ui": "vitest --ui"`.
- Run single test (after setup): `npm run test -- path/to/file.test.tsx` or `npx vitest path -t "case"`.
- Co-locate tests near components or under `tests/`; mock browser-only APIs (Clipboard, window) in Node.
- Prefer behavioral assertions over snapshotting large Mermaid/notes content.

## Project Map

- `src/app/`: routes/layout. `src/app/page.tsx` uses ISR 30s.
- `src/app/diagram/[id]/`: diagram editor (client-heavy); gate browser libs with mounted checks.
- `src/app/notes/`: notes layout + list; dynamic fetch with `getNotePage`; entry page shows empty-state; `[id]/page` loads note with `getNoteById` and renders `NoteWorkspace`; `dynamic = 'force-dynamic'` on layout and page to avoid caching.
- `src/app/api/`: API routes; keep server-only.
- `src/components/`: feature/layout components. Avoid touching `src/components/ui` unless fixing a shared UI bug.
- `src/lib/`: utilities, data access (`notes-data`), types, Zustand store.
- `data/`: SQLite db/backups; `public/`: static assets; `scripts/`: bootstrap + Prisma helpers (bootstrap must run before dev/build).
- `docs/`: repo docs; update this file and `docs/AI.md` if conventions change.
- `plan/`: planning docs; `plan/code-map.md` must stay current with structure and ownership.

## Styling

- Tailwind utilities; compose with `cn` from `src/lib/utils.ts`.
- Class order: layout → spacing → typography → colors → effects; keep classes readable.
- Respect design system: use Shadcn primitives and override via `className` rather than editing primitives.
- Support dark mode using semantic tokens (`bg-background`, `text-foreground`, `border-border`).
- Avoid generic purple/white defaults; pick intentional palettes when adding UI.

## State

- Use Zustand store setters; avoid direct mutation; prefer functional updates.
- Derive filtered/sorted arrays near usage to limit re-renders; supply stable keys (diagram id).

## React/Next Conventions

- Default to server components; add `'use client'` only when using state/effects/browser APIs.
- Gate browser-only libs (Mermaid, CodeMirror, react-resizable-panels, react-zoom-pan-pinch) with mounted checks (`useEffect`).
- Use async/await for data fetching; API routes return `NextResponse` with status codes; avoid accessing `window`/`document` in route handlers.
- Keep route handlers side-effect minimal; validate input; avoid long blocking writes; ensure directories exist before file writes (`fs.mkdir` recursive).
- Preserve `revalidate` settings (e.g., home uses ISR 30s) unless requirements change.
- Follow App Router naming (`page.tsx`, `layout.tsx`, `route.ts`).
- Type `params`/`searchParams` objects; avoid dynamic `any` route params.
- Notes routes: `NotesLayout` fetches first page server-side; `NotePage` awaits params via `params: Promise<{ id: string }>`; ensure types match this shape.

## Imports & Formatting

- Order imports: external libs, then `@/` absolute, then relative; separate groups with a blank line when edits are sizable.
- Use named exports; default exports only when Next requires (pages, route handlers).
- Type-only imports use `type` modifier (`import type { Diagram } from '@/lib/types';`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fantastic-Computing-Machine/atlantis](https://github.com/Fantastic-Computing-Machine/atlantis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
