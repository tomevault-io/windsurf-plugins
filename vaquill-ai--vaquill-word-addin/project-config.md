---
trigger: always_on
description: Vaquill for Microsoft Word - a Word task-pane add-in. Vite + React 18 + TypeScript SPA, Office.js (WordApi 1.6 floor), Supabase auth. Thin client over the existing Vaquill FastAPI backend.
---

# CLAUDE.md

Vaquill for Microsoft Word - a Word task-pane add-in. Vite + React 18 + TypeScript SPA, Office.js (WordApi 1.6 floor), Supabase auth. Thin client over the existing Vaquill FastAPI backend.

> This is a **separate repo** from the main Vaquill backend. The add-in performs no contract analysis, retrieval, or generation of its own. The legal intelligence lives in the backend; this repo reads the open document through Office.js, calls the backend, and applies results back as native Word tracked changes, comments, and content controls. It will not run with only an LLM API key.

## Quick Reference Commands

```bash
npm install
cp .env.example .env                 # set VITE_SUPABASE_URL and VITE_SUPABASE_ANON_KEY
npx office-addin-dev-certs install   # trusted HTTPS for localhost (one-time)
npm run dev                          # serve the task pane on https://localhost:3000
npm run sideload                     # load manifest.dev.xml into Word desktop
npm run sideload:stop
npm run type-check                   # tsc --noEmit
npm run build                        # tsc -b && vite build -> dist/
npm run lint                         # eslint
npm run validate:manifest            # validate manifest.xml
```

Requires Node 20+, a Microsoft 365 account, and Word (desktop or web). Deployment (Docker + hardened nginx, CSP, security headers) is in [DEPLOY.md](DEPLOY.md).

## Architecture

```text
Word (desktop / Mac / web)
  task pane (word.vaquill.ai)  --Office.js-->  the open document
        |
        |  Supabase JWT (Bearer) + SSE
        v
  Vaquill AI backend (api.vaquill.ai)   [required, unchanged except CORS]
```

Two hosted surfaces: the static task-pane SPA at `word.vaquill.ai`, and the unchanged backend at `api.vaquill.ai` reached over the same Supabase-JWT bearer auth and SSE contract the web app uses.

### The only required backend change

Add `https://word.vaquill.ai` (and `https://localhost:3000` for dev) to `CORS_ORIGINS` in the backend `app/core/config.py`. Because `CORS_ALLOW_CREDENTIALS` is true, the exact origin must match, so the pane must be served from `word.vaquill.ai`. `Authorization`, `Content-Type`, `X-Organization-ID`, and `X-Timezone` are already whitelisted. Everything else reuses existing endpoints.

## App Shell and Navigation

`src/App.tsx` renders seven primary tabs (`src/app/nav.tsx` `AppTab`): **home, review, draft, assistant, research, playbook, tools**. Home is a cockpit that routes into the others. Review is one hub with a sub-nav (`ReviewSub`: redlines / changes / citations / signoff). The document utilities (compliance, redact, fill, edit, transplant) fold under the single Tools launcher (`ToolKey`).

### Nav + intent bus (the cross-feature spine)

`src/app/nav.tsx` is the connective tissue. A lawyer's task crosses surfaces ("this clause is risky -> what's my position -> redline it"), so any surface can `navigate(tab, intent)` and hand the target a typed `AppIntent` (a pre-filled next step). The target view reads the intent on mount and calls `clearIntent()` once applied, so an intent fires exactly once. When adding cross-feature handoffs, extend `AppIntent` rather than wiring ad-hoc props.

The active-org change bumps `orgVersion`, which is the `key` on `app-body`, remounting data views (matters / drafts / playbooks / clients) so they refetch under the new org.

## Key Directories

| Directory | Purpose |
|-----------|---------|
| `src/office/` | All Office.js document I/O (redline apply, search anchoring, comments, content controls, export, custom XML, selection). The only place `Word.run` lives. |
| `src/api/` | Bearer `fetch`, SSE parser, and one module per backend surface (contract-review, drafting, playbooks, chat, authority, research, etc.). |
| `src/auth/` | Office Dialog API + Supabase PKCE login, in-memory session, refresh rotation. |
| `src/features/` | One folder per surface (review, draft, assistant, research, playbook, compliance, redact, fill, edit, transplant, governance, home, tools, integration). |
| `src/ui/` | Shared primitives (Combobox, OverflowMenu, ToolCard, progress, icons, tokens). White theme only. |
| `src/lib/` | Cross-cutting helpers (prefs, org, sections, severity, governance, hash, tiptap). |
| `src/config.ts` | Runtime config. `apiBase` / `appBase` fixed by build mode; Supabase URL + anon key injected at build. No secrets. |
| `docs/` | Internal design and research notes (not published in this repo). |

## Critical Gotchas (Office.js)

### Tracked-change author is read-only
Office.js cannot set the author of a tracked change. Every programmatic edit made while tracking is on is attributed to the signed-in Word user. When edits must be attributed to "Vaquill AI Contract Review" regardless of the user's identity, use the **server-side export path** (`POST /legal-tools/export-corrected`, inserted via `body.insertFileFromBase64`), not in-pane edits.

### Redline apply = TrackAll + tracked insertText

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Vaquill-AI/vaquill-word-addin](https://github.com/Vaquill-AI/vaquill-word-addin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
