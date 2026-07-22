---
trigger: always_on
description: (This file is also exposed as `CLAUDE.md` via a symlink so Claude Code loads it.)
---

# Project notes for AI agents

(This file is also exposed as `CLAUDE.md` via a symlink so Claude Code loads it.)

## Reference docs

Deeper format/architecture references live in [`doc/`](./doc/). Consult them before designing changes in the same area:

- [Architecture & code-layout reference](./doc/architecture.md) — the feature-folder convention, the Redux store wiring (`rootReducer.ts` / `processors.ts`), the **processor middleware** that owns all side effects, state persistence + rehydration, URL-hash sync, the global vs. per-feature i18n split, path aliases, and the layer/tool registries. **Read this before adding a feature, a side effect, persisted state, or a new menu/tool/modal — it saves re-reading the tree.**
- [Drawing feature export/import format mapping](./doc/drawing-export-mapping.md) — GPX/GeoJSON write+read for drawing points/lines/polygons, the `fm:*` lossless-shadow design, and the curated Garmin/OsmAnd icon dictionaries. **Read this before changing anything in `src/features/export/` or in the drawing-conversion processors.**
- [Track data: import, elevation, colorize, export](./doc/elevation-and-colorizers.md) — the shared elevation-acquisition layer (`src/shared/elevation.ts`: `fetchElevations`/`enrichElevations`/`densifyAlong`), the shared colorizers (`src/shared/colorizers/`, `isAvailable`-gated per feature), the render-only densified geometry that's never exported, the per-consumer elevation policy (route auto-fill vs. track prompt vs. tracking keep-recorded), and KML/KMZ/TCX import + lossless GeoJSON↔GPX transfer. **Read this before touching elevation, colorize, the elevation chart, or track-file import/export across routePlanner/trackViewer/tracking/mapFeaturesExport.**
- [SEO bot-prerender architecture](./doc/seo-prerender.md) — how crawlers get static HTML snapshots while humans get the SPA: the `sitemap-generator/` (homepages, layer/tool hub pages, per-feature POI pages), the nginx `/__prerender` routing, and the `rspack` `index-<lang>` variants. **Read this before changing `sitemap-generator/`, the sitemap/robots wiring, or the bot routing in the nginx vhost.**
- [Build & deploy gotchas](./doc/build-and-deploy.md) — the `DEPLOYMENT` env var (not `--mode`) selects the prod build, native CSS nesting must be downleveled via LightningCSS, and the nginx cache-header rules (entry HTML `no-store`, only hashed assets `immutable`). **Read this before changing `rspack.config.ts`, the CSS pipeline, or the nginx vhosts.**
- [UI conventions](./doc/ui-conventions.md) — what each react-bootstrap button `variant` means by role (`dark` = dismiss/close, `primary` = main CTA, `secondary` = default neutral action, `danger` = destructive, `outline-primary` = toggle, `warning` = upsell). **Read this before picking a `variant` for a new `<Button>`.**
- [Foreign-growth promotion roadmap](./doc/promotion-roadmap.md) — data-driven plan (from Matomo country/language/referrer cuts) for growing usage and premium conversions outside Slovakia: which markets to target (Italy, Poland, Hungary, Austria, …), which UI languages to add next (Slovenian, then French), and the phased outreach steps. **Read this before working on foreign promotion, deciding the next UI localization, or interpreting the audience analytics.**

## Keeping `llms.txt` in sync

[`src/static/llms.txt`](./src/static/llms.txt) is a hand-maintained, user-facing description of the app for AI assistants. It is **not** generated, so it drifts unless updated deliberately. When a change touches user-visible behavior, update the matching part of `llms.txt` in the same change set:

- **Map layers** (`src/shared/mapDefinitions.tsx`) — the "Layer registry" tables (ids, zoom levels, premium-from-zoom, credits, shortcuts, countries, base/overlay). This is the most drift-prone area.
- **Menu / tools / modals** — added, removed, or renamed main-menu items, tools, toolbars, keyboard shortcuts, URL hash params (`#show=…`, `#tool=…`, `layers=…`), or login/export providers. Cross-check labels against `src/translations/en.tsx`.

If a change makes `llms.txt` wrong but you can't fully fix it, say so explicitly rather than leaving it silently stale.

## Workflow

- **Do not commit until I explicitly approve.** Leave the working tree dirty at the end of a task and wait for "commit"/"looks good"/similar. This overrides any "commit when done" instinct.
- **When I tell you to commit, add yourself as co-author** via a `Co-Authored-By: Claude <noreply@anthropic.com>` trailer in the commit message.
- **Run `npx biome check --write <changed files>` after every change set**, before reporting completion. The project uses Biome with import-order enforcement, so manually arranging imports is wasted effort. Pair with `npx tsc --noEmit -p .` for type safety.
- **Comments describe current behavior, not history.** Write comments, JSDoc, and test names in the present tense about what the code does now. Don't narrate how it used to work or what a refactor changed (no "previously…", "no longer…", "used to live under…", "dropped from…"). Change history belongs in the commit/diff, not in the source. (A backlog/changelog entry like `TODO.md` may use past tense — it's an explicit record of work done.)

## Translations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FreemapSlovakia/freemap-v3-react](https://github.com/FreemapSlovakia/freemap-v3-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
