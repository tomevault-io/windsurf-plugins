---
trigger: always_on
description: Mediant is a minimal Org-mode parser and agenda viewer. It parses a focused subset of Org syntax and renders a responsive rolling 7-day agenda in HTML/CSS. It runs in two modes: a **static mode** where users paste Org content into a textarea (localStorage-backed), and a **server mode** where a local Node CLI (`mediant <file.org>`) serves the UI and streams the configured Org file over `/api/source` + SSE. No framework dependencies.
---

# AGENTS.md

## Project overview

Mediant is a minimal Org-mode parser and agenda viewer. It parses a focused subset of Org syntax and renders a responsive rolling 7-day agenda in HTML/CSS. It runs in two modes: a **static mode** where users paste Org content into a textarea (localStorage-backed), and a **server mode** where a local Node CLI (`mediant <file.org>`) serves the UI and streams the configured Org file over `/api/source` + SSE. No framework dependencies.

## Architecture

Three clearly separated stages — do not collapse them:

```
  .org file → Parser (org/) → Agenda (agenda/)    → UI (ui/)
              OrgEntry[]       AgendaWeek            HTML/CSS
                               DeadlineItem[]
                               OverdueItem[]
                               SomedayItem[]
```

- **Parser output types** (`src/org/`) reflect Org source faithfully
- **Agenda types** (`src/agenda/`) reflect UI needs (render categories, 7-day structure)
- Classification into display categories happens at the agenda stage, never during parsing

## Key source files

| File | Responsibility |
|---|---|
| `src/org/timestamp.ts` | Timestamp parsing, Date conversion, recurrence expansion, per-occurrence exception application. **Only** module that does date arithmetic. Exports both `expandRecurrences()` (pure base series) and `expandOccurrences()` (base series with exceptions applied). |
| `src/org/parser.ts` | Line-by-line Org parser → `OrgEntry[]`. Delegates all timestamp work to `timestamp.ts`. Reads `:EXCEPTION-<date>:`, `:EXCEPTION-NOTE-<date>:`, and `:SERIES-UNTIL:` keys inside `:PROPERTIES:` drawers; every other drawer and every other property key is skipped. |
| `src/org/model.ts` | Parser output types: `OrgEntry` (including `seriesUntil`), `OrgPlanning`, `TodoState`, `Priority`, `CheckboxItem`, `RecurrenceOverride`, `RecurrenceException`. |
| `src/org/drawer.ts` | Property-drawer text mutation helpers (`upsertProperty`, `removeProperty`). Operate on raw source strings; preserve key order and other drawer content; used by the edit panel to write exception properties. |
| `src/org/sourceEdit.ts` | Pure raw-source mutation helpers used by UI actions: replace an entry block while preserving body text, toggle TODO/DONE, advance repeating timestamps when marking done, toggle checkbox state, and update progress cookies. |
| `src/agenda/model.ts` | Agenda/render types: `AgendaItem`, `AgendaDay`, `AgendaWeek`, `DeadlineItem`, `OverdueItem`, `SomedayItem`, `RenderCategory`, `AgendaItemOverride`. |
| `src/agenda/generate.ts` | 7-day generation from a start date, recurrence expansion with exceptions (bounded to requested range), classification, sorting, overdue/someday collection. |
| `src/dateLabels.ts` | Shared English day/month labels for agenda headers and date formatting. Keep display labels centralized here instead of duplicating arrays. |
| `src/ui/render.ts` | DOM rendering from `AgendaWeek` + `DeadlineItem[]` + `OverdueItem[]`. Renders the per-occurrence override chip, instance note, active tag-filter state, and tag color-mode UI. |
| `src/ui/tagColors.ts` | Dynamic tag color management. Auto-assigns from palette, persists in localStorage. |
| `src/ui/notifications.ts` | Browser notification preference, permission request, and timer scheduling for timed events happening today. Notifications fire 1 hour before the start time and are rescheduled on render. |
| `src/ui/style.css` | All styles. CSS grid layout with content-width time column. |
| `src/main.ts` | Entry point. Probes `/api/source` on boot; if present, enters server mode (hydrates from the server, subscribes to `/api/events` for external file changes). Otherwise shows the textarea input screen backed by localStorage. Owns global keyboard shortcuts, tag-filter state, tag color mode, quick-capture overlay, add-item & edit-item panels, and the "This occurrence" section that writes exception properties via the drawer helpers. |
| `server/cli.mjs` | Node CLI + HTTP server. `mediant <file.org> [--port N] [--daemon]`. Serves `dist/` plus `GET/PUT /api/source` (with `If-Match` version checks) and `GET /api/events` SSE backed by `fs.watch`. Node built-ins only, no deps. |
| `elisp/mediant-org-agenda.el` | Optional Emacs Org agenda integration. Global minor mode that runs on `org-agenda-finalize-hook`, reads Mediant exception properties from source headings, filters cancelled/cutoff occurrences, inserts moved synthetic agenda lines, and renders exception notes. |
| `elisp/mediant-org-agenda-test.el` | ERT tests for the optional Org agenda integration. Uses temporary Org files and real `org-agenda-list` generation to verify exception display behavior. |

## Commands

```sh
npm test              # run all tests (vitest)
npm run test:watch    # vitest in watch mode
npx vite              # dev server (serves index.html)
npm run build         # build dist/ for the server to serve

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jovlang/Mediant](https://github.com/Jovlang/Mediant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
