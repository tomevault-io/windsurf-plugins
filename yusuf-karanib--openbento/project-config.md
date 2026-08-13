---
trigger: always_on
description: This file gives coding agents the working context for OpenBento. Use it with
---

# AGENTS.md

This file gives coding agents the working context for OpenBento. Use it with
`CLAUDE.md`, `replit.md`, and the code itself; the code is the source of truth
when docs drift.

## What OpenBento Is

OpenBento is a free, ad-supported, bento-style Mission Control dashboard for
monitoring live streams, personal notes, weather, markets, feeds, utilities, and
small interactive widgets. Users can customize a 12-column dashboard, resize and
move widgets, create multiple dashboard pages, theme the workspace, and
optionally sign in for cross-device cloud sync.

The product is not a SaaS paywall app. Guests can use the dashboard; signing in
only enables sync and library persistence.

## Product Principles

- Keep the dashboard useful immediately for guests.
- Preserve layout and widget state reliably; avoid migrations that discard user
  data from localStorage or cloud sync.
- Prefer focused, practical widgets over broad feature surfaces.
- Make widgets feel native to the dashboard grid: compact, resizable, legible,
  and safe in fullscreen/cast contexts.
- Keep OpenBento fully free and ad-supported. Do not add premium tiers,
  payment processing, locked widgets, usage limits, or paywall logic.
- Ask before major product or data-model changes.

## Repo Structure

- `client/` - React 18 + Vite SPA using Wouter, Tailwind CSS, `@dnd-kit`, and
  Radix/lucide UI pieces.
- `client/src/App.tsx` - top-level providers and route table. It re-exports
  widget types for older imports.
- `client/src/dashboard/dashboard-shell.tsx` - dashboard state owner: auth
  wiring, DnD, add/edit/delete callbacks, localStorage persistence, and cloud
  sync hook usage.
- `client/src/pages/dashboard.tsx` - visual dashboard surface: 12 x 6 grid,
  widget chrome, resize handles, header controls, fullscreen, themes, and
  video-specific controls.
- `client/src/widgets/` - widget data model, renderer registry, and most widget
  implementations.
- `client/src/components/widget-sidebar.tsx` - Block Library, stream presets,
  widget templates, custom widget install flow, and personal stream library.
- `client/src/lib/command-palette-helpers.ts` - command palette command builder,
  including Add Widget commands from widget templates.
- `server/` - Express 5 API server. The same port serves APIs and built client
  assets.
- `server/routes.ts` - main API route file.
- `server/storage.ts` - Drizzle-backed storage operations.
- `shared/` - shared Drizzle models, dashboard page helpers, themes, widget SDK
  protocol, and shared types.
- `shared/dashboard-pages.ts` - pure multi-page dashboard state helpers and
  localStorage key constants.
- `shared/models/` - Drizzle schemas for auth, dashboards, streams, channels,
  feedback, cast, and related records.
- `mobile/` - Expo mobile companion app.

Path aliases:

- `@/` maps to `client/src/`
- `@shared/` maps to `shared/`

## Main Commands

```bash
npm run dev        # Start Express + Vite dev server on port 5000
npm run build      # Build client to dist/public and server to dist/index.cjs
npm run start      # Run production server
npm run check      # TypeScript type checking, no emit
npm run db:push    # Push Drizzle schema changes to PostgreSQL
```

No test runner is configured. Before committing, always run:

```bash
npm run check
npm run build
```

## Widget Architecture

Widget state is persisted as JSON blobs inside dashboard pages. The client-side
widget model lives in `client/src/widgets/shared.tsx`:

- `WidgetType` is the union of valid widget type strings.
- `Widget` is the persisted shape for base layout fields plus per-widget fields.
- Widget type strings must stay lowercase and must exactly match sidebar
  template strings and registry keys.

Rendering is split like this:

- `client/src/widgets/registry.tsx` maps `Widget.type` to a renderer component.
- `client/src/widgets/widget-renderer.tsx` dispatches through the registry and
  shows an Unknown Widget tile for missing types.
- Most widgets live in one file under `client/src/widgets/*-widget.tsx`.
- `video` is special: the registry returns `false`, and
  `client/src/pages/dashboard.tsx` renders `VideoWidget` inline because it needs
  iframe refs, seek mode state, platform controls, and stream-healing behavior.

The Block Library has two relevant sources in
`client/src/components/widget-sidebar.tsx`:

- `WIDGET_TEMPLATES` powers drag/drop templates and command palette Add Widget
  commands.
- `availableWidgets` powers the visible sidebar cards.

If you add a widget, update both when the widget should be visible in both flows.

## How To Add A New Widget Safely

Use the smallest set of files possible:

1. Add the widget type and any persisted fields to
   `client/src/widgets/shared.tsx`.
2. Create the component in `client/src/widgets/<name>-widget.tsx`.
3. Register it in `client/src/widgets/registry.tsx`.
4. Add it to `WIDGET_TEMPLATES` and `availableWidgets` in
   `client/src/components/widget-sidebar.tsx`.
5. Add default values in `addWidget` inside
   `client/src/dashboard/dashboard-shell.tsx` only if the widget needs defaults
   beyond the base widget fields.
6. If the widget should appear in starter experiences, update
   `client/src/data/starter-packs.ts`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Yusuf-Karanib/OpenBento](https://github.com/Yusuf-Karanib/OpenBento) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
