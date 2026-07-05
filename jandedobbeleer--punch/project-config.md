---
trigger: always_on
description: generates a branded PDF (styled after the IT depends offering documents —
---

# Tempo — Agent Instructions

Tempo is a personal time-tracking single-page app. This file orients any
coding agent (GitHub Copilot, etc.) working in this repository.

## Architecture

- **Frontend**: React 19 + TypeScript + Vite SPA, hosted as an **Azure Static
  Web App**. Deployed from repo root (`app_location: /`, `output_location:
  dist`) — there is no `react-app/` subfolder, everything lives at repo root.
- **Backend**: Azure Functions (Node/TypeScript, managed Functions integrated
  with the Static Web App) in `api/`, providing the `/api/*` routes.
- **Storage**: a single Azure Storage account (`<your-storage-account>`) with two
  blob containers:
  - `state` — one JSON blob (`state/tempo.json`) holding the entire app state
    (`customers[]`, `projects[]`, `entries[]`). Synced via optimistic
    concurrency using the blob's ETag (`If-Match` header on writes; a 412
    response means someone else wrote first — client refetches and re-applies).
  - `attachments` — files attached to time entries (e.g. receipts),
    addressed as `<entryId>/<attachmentId>`. Upload/download use short-lived
    SAS URLs issued by the Functions API so file bytes never pass through the
    Function itself.
- **Auth**: GitHub sign-in via Azure Static Web Apps built-in auth
  (`/.auth/login/github`). Access is restricted to a single invited identity
  assigned the custom `owner` role (via `az staticwebapp users invite`, not
  via the Standard-plan password-protection feature). `staticwebapp.config.json`
  gates **every** route (`/*` and `/api/*`) behind `allowedRoles: ["owner"]`.
  There is intentionally no multi-tenant/multi-user support.
- **CI/CD**: `.github/workflows/deploy.yml` builds and deploys on every push
  to `main` (and manages PR preview environments) via
  `Azure/static-web-apps-deploy@v1`. It requires the
  `AZURE_STATIC_WEB_APPS_API_TOKEN` repo secret (the SWA deployment token).

This is a **single-user prototype**. Don't add multi-tenant auth, database
migrations, or backwards-compat data migration logic unless explicitly asked
— simplicity is preferred over generality at this stage.

## Repository layout

```
src/                      # Frontend SPA (Vite root)
  types.ts                # Single source of truth for domain + view-model shapes
  hooks/useTempoState.ts  # Core state machine: CRUD, view-model building, sync, attachments
  lib/store.ts            # All backend I/O: fetchState/saveState (ETag sync),
                           #   uploadAttachment/getAttachmentDownloadUrl/deleteAttachment,
                           #   plus localStorage cache helpers. No GitHub/Gist code — removed.
  lib/dates.ts, format.ts, rates.ts
  components/             # Presentational components (Sidebar, Modal, views per page, Settings)
api/                       # Azure Functions API
  src/functions/state.ts       # GET/PUT /api/state (ETag concurrency)
  src/functions/attachments.ts # POST /api/attachments (SAS upload ticket),
                                #   GET/DELETE /api/attachments/{entryId}/{attachmentId}
  src/blobClient.ts        # Shared BlobServiceClient factory (connection string or
                            #   managed identity + DefaultAzureCredential)
  src/auth.ts              # Reads SWA's x-ms-client-principal header; requireOwner() guard
  local.settings.json.example  # Copy to local.settings.json for local `func start`
                                # (never commit local.settings.json — it's gitignored)
staticwebapp.config.json   # Route auth rules (owner-only), SPA fallback, API runtime version
e2e/                       # Playwright end-to-end specs
.github/workflows/deploy.yml  # SWA CI/CD (build + deploy on push to main)
```

## Feature authoring pattern

Every feature that changes behaviour (not just style) follows this three-step
sequence — **in order**, to keep TypeScript happy throughout:

1. **`src/types.ts`** — add/extend the domain type and the view-model
   interface for the affected component(s). Both sides of the state boundary
   must agree before either compiles cleanly.

2. **`src/hooks/useTempoState.ts`** — implement the business logic and extend
   the view-model builder (`useMemo` block) to populate the new fields. This
   file is the *only* place that reads or mutates domain state; components
   never import from `store.ts` or access raw domain arrays.

3. **The component** (`src/components/*.tsx`) — consume the new view-model
   props. Accept data as props, render it, call callbacks. Done.

Always run `npx tsc -b` after each step and fix errors before moving to the
next. This prevents misleading type errors that look like step-3 bugs but are
actually step-1 omissions.

## Styling conventions

Structural styles (dimensions, flex, colour, typography, spacing) live as
**inline `style` props** (`CSSProperties` objects) in the TSX files and in
`useTempoState.ts`. This is consistent throughout the codebase — do not
introduce CSS modules, styled-components, or Tailwind.

Anything requiring `@media`, `:hover`, `:active`, `transition`, `animation`,
or `z-index` stacking belongs in **`src/index.css`** as a named CSS class.
Components opt in via `className`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JanDeDobbeleer/punch](https://github.com/JanDeDobbeleer/punch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
