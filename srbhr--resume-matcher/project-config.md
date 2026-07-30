---
trigger: always_on
description: > Frontend deep-dive for Claude Code. Read the repo-root [`.claude/CLAUDE.md`](../../.claude/CLAUDE.md) and [`docs/agent/README.md`](../../docs/agent/README.md) first for project-wide context. This file goes deeper on the Next.js app only.
---

# CLAUDE.md - Frontend (apps/frontend)

> Frontend deep-dive for Claude Code. Read the repo-root [`.claude/CLAUDE.md`](../../.claude/CLAUDE.md) and [`docs/agent/README.md`](../../docs/agent/README.md) first for project-wide context. This file goes deeper on the Next.js app only.

**Stack:** Next.js 16 (App Router, Turbopack) · React 19 · TypeScript (strict) · Tailwind CSS v4 · no UI framework (hand-rolled `components/ui`). Import alias `@/*` → `apps/frontend/*`.

---

## Route / Page Map

App Router under `app/`. A `(default)` route group wraps the main app in providers; `print/*` is provider-free (server-rendered for headless-Chromium PDF capture).

| Route | File | Type | Purpose |
|-------|------|------|---------|
| `/` | `app/(default)/page.tsx` | Server | Landing — renders `<Hero/>` |
| `/dashboard` | `app/(default)/dashboard/page.tsx` | Client | Resume list, upload, delete, retry, status grid |
| `/builder` | `app/(default)/builder/page.tsx` | Client wrapper → `components/builder/resume-builder.tsx` | Master-resume editor (forms, drag-drop sections, templates, AI regenerate, cover letter / outreach) |
| `/tailor` | `app/(default)/tailor/page.tsx` | Client | Paste JD → preview/confirm tailored resume (diff modal) |
| `/tracker` | `app/(default)/tracker/page.tsx` | Client | Kanban application tracker — 7-column board (drag/drop, bulk ops, manual add) |
| `/settings` | `app/(default)/settings/page.tsx` | Client | LLM provider/model/key, per-provider API keys, features, prompts, language, reset DB |
| `/resumes/[id]` | `app/(default)/resumes/[id]/page.tsx` | Client | View one resume, download PDF, rename, enrichment modal |
| `/print/resumes/[id]` | `app/print/resumes/[id]/page.tsx` | **Server** | Print-only resume render for PDF (reads `searchParams` for template settings + `lang`) |
| `/print/cover-letter/[id]` | `app/print/cover-letter/[id]/page.tsx` | **Server** | Print-only cover-letter render for PDF |

`app/layout.tsx` (root) wires fonts (Geist + Space Grotesk) and global CSS. `app/(default)/layout.tsx` nests providers: `StatusCacheProvider` → `LanguageProvider` → `ResumePreviewProvider` → `LocalizedErrorBoundary`.

> Most pages are `'use client'`. The `print/*` pages are intentionally server components and fetch from the backend directly via `API_BASE` + `lib/i18n/server.ts` (`translate`). Do not add `'use client'` to them.

---

## Directory Layout

```
app/                 # routes (see table)
components/
  ui/                # primitives: button, input, textarea, dialog, dropdown,
                     #   card, retro-tabs, toggle-switch, confirm-dialog,
                     #   rich-text-editor (Tiptap), link-dialog, label
  builder/           # builder page UI + forms/ (per-section form components)
  dashboard/         # resume list/card, upload dialog
  tailor/            # diff-preview-modal
  tracker/           # kanban-board, kanban-column, application-card,
                     #   card-detail-modal, bulk-action-bar,
                     #   manual-add-application-dialog, reorder.ts (pure planMove)
  enrichment/        # AI enrichment wizard modal/steps
  resume/            # resume render templates (single/two-column, modern) + styles/*.module.css
  preview/           # paginated A4/Letter preview (use-pagination.ts)
  home/              # hero, swiss-grid
  settings/          # api-key-menu
  common/            # error-boundary, resume_previewer_context
lib/
  api/               # backend client (see Data Flow)
  i18n/              # translation engine (see i18n)
  context/           # status-cache, language-context
  utils/             # download, html-sanitizer, keyword-matcher, section-helpers
  types/             # template-settings, lucide.d.ts
  config/version.ts  # APP_VERSION / codename
  constants/page-dimensions.ts
hooks/               # use-file-upload, use-regenerate-wizard, use-enrichment-wizard
i18n/config.ts       # locale list + names/flags (NOTE: distinct from lib/i18n)
messages/            # en/es/zh/ja/pt-BR JSON (see i18n)
tests/               # vitest (see Testing)
```

---

## Data Flow (page → hook → lib/api → backend)

All backend calls go through **`lib/api/`** — never call `fetch` to the backend directly from a component.

- `lib/api/client.ts` — single source of truth. Exports `apiFetch / apiPost / apiPatch / apiPut / apiDelete`, `API_URL`, `API_BASE`, `getUploadUrl()`.
  - Base URL: `NEXT_PUBLIC_API_URL` (default `'/'`) → `API_BASE` becomes `/api/v1`. On the **server** a `/`-relative base is rewritten to `http://127.0.0.1:8000/api/v1` (`INTERNAL_API_ORIGIN`); browser uses the relative path (proxied by `next.config.ts` rewrites to `BACKEND_ORIGIN`).
  - Default request timeout **240_000ms** (matches backend `wait_for` hard limit). `AbortError` → friendly "Request timed out" message.
- `lib/api/resume.ts` — resumes/jobs: upload, improve / improve.preview / improve.confirm, fetch, list, update (PATCH), PDF URLs + blob download, delete, cover-letter / outreach generate+update, rename, retry-processing, fetch JD.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [srbhr/Resume-Matcher](https://github.com/srbhr/Resume-Matcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
