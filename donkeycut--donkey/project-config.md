---
trigger: always_on
description: Donkey is a video editor. Donkey Cut runs in the browser; the Mac app is a menu bar app
---

# Agent Guide

Donkey is a video editor. Donkey Cut runs in the browser; the Mac app is a menu bar app
whose only job is to let that page use the Mac's hardware — the local Cut engine (encoding,
storage, speech-to-text) and screen recording.

`docs/` holds supported product behavior and engineering guidance. Start with `docs/README.md`
when changing supported behavior.

Never infer semantic intent by string matching raw user input. Do not add phrase lists, prefixes, suffixes, regexes, app-name checks, greeting/help classifiers, or other natural-language command-text matching to decide what the user wants. Raw user text has too many variations to handle reliably. Pass the turn through an LLM or another typed model/runtime boundary first, get structured output, then do deterministic matching only on that structured output or on non-semantic technical fields.

## Performance

Performance comes first. Everything we ship has to be fast on the machines people edit on, and a change that makes the preview, the timeline, or the app slower is a regression whatever else it adds.
`docs/guides/performance.md` is the guide: the frame budget, the techniques, and how a change is measured.

- Work scales with what is on screen and what the person is doing, never with the size of the project. Draw and decode the visible part, queue the rest behind it, and skip what nothing shows.
- Keep the main thread and the frame loop clear: one animation frame per tick, no synchronous decodes or layout reads inside a hot path, no allocation per frame.
- Every cache is bounded in bytes through the memory budget (`site/src/cut/lib/memoryBudget.ts`), and every reader, decoder and canvas is closed by the code that opened it.
- Measure. A preview or timeline change runs the perf evals (`npm run eval:cut-perf`, `eval:cut-perf-lowend`) before it ships, and the summary says what moved.

## Site Project

Before changing `site/` UI, routes, API handlers, or data access patterns:

- Read the relevant Next.js guide in `site/node_modules/next/dist/docs/`; this version may differ from your training data.
- Read the applicable site guidance in `docs/guides/`.
- Do not hand-write SQL migrations.
- Every database read and write goes through the Prisma client API. Never use `$queryRaw` or `$executeRaw`; an atomic counter is `updateMany` with `increment` guarded by a `where` on the current value, checked through its count.
- Do not run database migrations, including `prisma migrate`, `prisma db push`, or any command that applies schema changes to Supabase or another database.
- Keep Prisma table/model definitions out of `site/prisma/schema.prisma`. Put tables in logically grouped sibling `.prisma` files under `site/prisma/`; reserve `schema.prisma` for shared Prisma configuration such as generator and datasource blocks.
- Treat `/prototype`, "the prototype route", or route-shaped prototype requests as work on the Next.js route under `site/`, not as a repository-root `prototype/` directory.

## Cut Surfaces

Every Cut change has to hold on every surface, and the plan for it says how:

- **Three residencies.** A project lives in the **browser** (OPFS in the page), on this **Mac** (the Bun engine inside the app, with the bundled command-line tools), or in the **cloud** (Postgres doc + R2 media, the work done by the container worker). People run all three — a Mac with or without the app, any browser, a cloud project — so a change holds in each of them. Work through the backend seam in `site/src/cut/lib/backend/`. Where one residency lacks the machinery for a job, hand the job to one that has it: the browser shelf imports links through the cloud worker, and the engine falls back to the same worker when its own tools come back empty-handed.
- **Headed and headless.** Whatever the tab can do, the Bun engine and the worker runner can do: chat tools, rendering, media reads. Headless installs browser primitives — canvas, decoders, Web Audio, fonts — behind narrow seams (`lib/raster.ts`, the frame sink in `lib/mediaRead.ts`, the font installer in `lib/fontAssets.ts`, the kit's `surface.ts`) so one implementation serves both; reach for those seams before writing a second path.
- Allocate canvases and decode images through the raster seam. Direct `document`, `window`, `FileReader`, `createImageBitmap`, or `FontFace` use in `site/src/cut/lib/` or `packages/effects-kit/` breaks a job.
- When a surface genuinely cannot carry a feature, give it a fallback and say so in the summary and the guide.
- **ChatGPT is a client too.** The MCP adapter under `site/src/clients/chatgpt/` runs the same project tool catalog as headless worker jobs, so a tool or schema change has to hold there as well. `docs/guides/chatgpt-app.md` is the guide. Every `embed=chatgpt` gate in the editor is deliberate — the card hides the generation tabs, Donkey's own chat, Share, Record and the credit pills, so leave them out of the frame.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DonkeyCut/Donkey](https://github.com/DonkeyCut/Donkey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
