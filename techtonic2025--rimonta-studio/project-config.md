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

## Site Project

Before changing `site/` UI, routes, API handlers, or data access patterns:

- Read the relevant Next.js guide in `site/node_modules/next/dist/docs/`; this version may differ from your training data.
- Read the applicable site guidance in `docs/guides/`.
- Do not hand-write SQL migrations.
- Do not run database migrations, including `prisma migrate`, `prisma db push`, or any command that applies schema changes to Supabase or another database.
- Keep Prisma table/model definitions out of `site/prisma/schema.prisma`. Put tables in logically grouped sibling `.prisma` files under `site/prisma/`; reserve `schema.prisma` for shared Prisma configuration such as generator and datasource blocks.
- Treat `/prototype`, "the prototype route", or route-shaped prototype requests as work on the Next.js route under `site/`, not as a repository-root `prototype/` directory.

## Cut Surfaces

Every Cut change has to hold on all four surfaces, and the plan for it says how:

- **Three residencies.** A project lives in the **browser** (OPFS in the page), on this **Mac** (the Bun engine inside the app, with the bundled command-line tools), or in the **cloud** (Postgres doc + R2 media, the work done by the container worker). People run all three — a Mac with or without the app, any browser, a cloud project — so a change holds in each of them. Work through the backend seam in `site/src/cut/lib/backend/`. Where one residency lacks the machinery for a job, hand the job to one that has it: the browser shelf imports links through the cloud worker, and the engine falls back to the same worker when its own tools come back empty-handed.
- **Headed and headless.** Whatever the tab can do, the Bun engine and the worker runner can do: chat tools, rendering, media reads. Headless installs browser primitives — canvas, decoders, Web Audio, fonts — behind narrow seams (`lib/raster.ts`, the frame sink in `lib/mediaRead.ts`, the font installer in `lib/fontAssets.ts`, the kit's `surface.ts`) so one implementation serves both; reach for those seams before writing a second path.
- Allocate canvases and decode images through the raster seam. Direct `document`, `window`, `FileReader`, `createImageBitmap`, or `FontFace` use in `site/src/cut/lib/` or `packages/effects-kit/` breaks a job.
- When a surface genuinely cannot carry a feature, give it a fallback and say so in the summary and the guide.
- **The AI chat drives it too.** A functional change ships with its chat surface: a tool the assistant can call, and descriptions/system-prompt lines that teach the new capability. Derive tool schemas and prompt text from the same exported constants the UI uses (style id lists, model registries) so the catalog updates itself; check the tool files beside the component (`*.tools.ts`) and `site/src/cut/server/ai/catalog.ts`.
- The chat surface is kept true, both directions. Removing or reshaping a feature means deleting its tool and its prompt/skill mentions in the same change — grep the catalog, the `*.tools.ts` files, the skills library, and `lib/aiTools.ts` for the old names, ids, and parameters. A tool that describes behavior the code no longer has, offers an option that no longer exists, or is missing a setting the UI gained is a bug: the model calls what the catalog teaches.

## Working Rules

- Do not touch repository-root `prototype/` unless the user explicitly asks for that filesystem path. By default, assume requested product changes are for the Mac app or the site/landing page.
- Ask before creating any new plan document.
- When writing or editing any engineering doc under `docs/`, follow `docs/guides/eng-doc-style.md`.
- Write straight up — in prompts, docs, commits, code comments, summaries, and UI copy. State what a thing is, once, and stop. Never frame it against what it is not: no "X, not Y", no "X rather than Y", no "instead of Z", no "…, which is exactly what not to do". Cut filler.
- Keep replies short and action-oriented. For implementation questions, give the recommendation first, then one to three short bullets on why; when the answer is obvious, just say what to do. Skip long explanations, caveats, and "one last thing" sections; flag a real blocker or risk with "One issue:" and explain it briefly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [techtonic2025/RiMonta-Studio](https://github.com/techtonic2025/RiMonta-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
