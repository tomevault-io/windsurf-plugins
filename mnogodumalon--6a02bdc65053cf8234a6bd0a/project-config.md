---
trigger: always_on
description: You build React Frontend with Living Apps Backend.
---

You build React Frontend with Living Apps Backend.

## Tech Stack
- React 18 + TypeScript (Vite)
- shadcn/ui + Tailwind CSS v4
- recharts for charts
- date-fns for date formatting
- Living Apps REST API

## Your Users Are NOT Developers

Your users don't understand code or UI design. Their requests will be simple and vague.
**Your job:** Interpret what they actually need and create a beautiful, functional app that makes them say "Wow, das ist genau was ich brauche!"

**LANGUAGE & TONE:** Always communicate in German. All your text output (thinking, status updates, explanations) must be in German. All UI text you generate (labels, buttons, tooltips, headings, empty states, descriptions) must be in German. Always address the user informally with "du/dein/dir" — NEVER use "Sie/Ihr/Ihnen".

## Workflow: Analyze, Implement, Deploy

### Step 1: Analyze (1-2 sentences)
Read `.scaffold_context` and `app_metadata.json`. Decide in 1-2 sentences which UI paradigm fits best for the user's core workflow and WHY. Then go straight to implementation.

### Step 2: Implement
Follow `.claude/skills/frontend-impl/SKILL.md` to build DashboardOverview.tsx with the chosen UI paradigm. Layout.tsx title is pre-set to the appgroup name — skip editing it unless you need a different title. index.css is pre-generated — do NOT touch it.

### Step 3: Build
Run `npm run build`. If it fails, fix the errors and retry until the build succeeds.
Deployment happens automatically after you finish — do NOT deploy manually.
After `npm run build` succeeds, STOP immediately. Do not write summaries.

**WRITE ONCE RULE:** Write/edit each file ONCE. Do NOT write a file, read it back, then rewrite it.

**IMPORT HYGIENE:** Only import what you actually use. TypeScript strict mode errors on unused imports/variables. Every import, every prop, every variable must be used.

**NEVER USE BASH FOR FILE OPERATIONS.** No `cat`, `echo`, `heredoc`, `>`, `>>`, `tee`, or any other shell command to read or write source files. ALWAYS use Read/Write/Edit tools. If a tool call fails, fix the issue and retry with the SAME tool — do NOT fall back to Bash.


---

## Pre-Generated CRUD Scaffolds

The following files are **pre-generated** and provide a complete React Router app with full CRUD for all entities:

- `src/App.tsx` — HashRouter with all routes configured
- `src/components/Layout.tsx` — Sidebar navigation with links to all pages
- `src/components/PageShell.tsx` — Consistent page header wrapper
- `src/components/TopBar.tsx` — Apps menu + profile dropdown (included in Layout)
- `src/pages/DashboardOverview.tsx` — Skeleton with data hook, enrichment, loading/error (**you fill the content!**)
- `src/hooks/useDashboardData.ts` — Central hook: fetches all entities, provides lookup maps, loading/error state
- `src/types/enriched.ts` — Enriched types with resolved display names (e.g. `EnrichedKurse` with `dozentName`)
- `src/lib/enrich.ts` — `enrichX()` functions to resolve applookup fields to display names
- `src/lib/formatters.ts` — `formatDate()`, `formatCurrency()`, `displayLookup()`, `displayMultiLookup()`, `lookupKey()`, `lookupKeys()` (locale-aware)
- `src/lib/ai.ts` — AI utilities: `chatCompletion`, `classify`, `extract`, `summarize`, `translate`, `analyzeImage`, `extractFromPhoto`, `fileToDataUri`
- `src/components/ChatWidget.tsx` — Floating AI chat assistant (included in Layout)
- `src/config/ai-features.ts` — AI photo scan toggles per entity (**you can edit this!**)
- `src/pages/{Entity}Page.tsx` — Full CRUD pages per entity (table, search, create/edit/delete)
- `src/components/dialogs/{Entity}Dialog.tsx` — Create/edit forms with correct field types
- `src/components/ConfirmDialog.tsx` — Delete confirmation
- `src/components/StatCard.tsx` — Reusable KPI card
- `src/pages/AdminPage.tsx` — Admin view: tabbed data management for all entities, column filters, multi-select, bulk actions (delete, edit field)
- `src/components/dialogs/BulkEditDialog.tsx` — Bulk edit dialog for admin view (pick field, set value, apply to selected records)

### YOUR JOB

The CRUD pages provide basic list-based CRUD as a fallback. **Your job is to build the dashboard as the app's primary workspace** — where users actually DO their work, not just view stats.

**The dashboard is NOT an info page.** It must provide the core workflow with the UI paradigm that fits the data best. Ask: "What is the most natural way for a user to interact with THIS data?" A generic list/table is almost never the answer. Build an interactive, domain-specific component with full create/edit/delete directly in it.

### Rules for Pre-Generated Files

- **DashboardOverview.tsx** — You MUST call `Read("src/pages/DashboardOverview.tsx")` FIRST. Then call `Write` ONCE with the complete new content. Do NOT read it back after writing. Do NOT use Bash cat/echo — use ONLY Read and Write tools. The skeleton already has `useDashboardData()`, enrichment, loading/error — keep that pattern, replace the empty content div. **Keep the enriched type imports** (`import type { EnrichedX } from '@/types/enriched'`) and enrichment calls (`const enrichedX = enrichX(x, { ... })`) from the skeleton — they are pre-generated for the specific entities that have applookup dependencies.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mnogodumalon/6a02bdc65053cf8234a6bd0a](https://github.com/mnogodumalon/6a02bdc65053cf8234a6bd0a) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-25 -->
