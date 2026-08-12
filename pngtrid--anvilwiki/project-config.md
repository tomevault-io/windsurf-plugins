---
trigger: always_on
description: Workspace instructions for ZCode agents working on AnvilWiki.
---

# AGENTS.md

Workspace instructions for ZCode agents working on AnvilWiki.

## Repository Purpose

AnvilWiki is an **open-source (MIT) game wiki site template** built with **Astro 5 + Cloudflare Pages**. It is a static-first Astro setup that deploys to Cloudflare with zero adapters and enjoys free unlimited bandwidth.

Goal: let beginners deploy a game wiki site to Cloudflare Pages for free (unlimited bandwidth) in ~30 minutes, with strong SEO, i18n, and ad-monetization built in.

**Status (as of 2026-08-11)**: Planning stage. Only `README.md` + `docs/PRD.md` exist. No code yet. Code MVP starts after PRD review.

## Read These First

- **`docs/PRD.md`** — the single source of truth for architecture, data models, module design, and roadmap. **Read before any code change.** 15 chapters + 3 appendices.
- `README.md` — project pitch + quick start (Chinese + English).

## Intended Tech Stack (verified, as of 2026-08-11)

| Layer       | Choice                                          | Notes                                                                                                                                                                                               |
| ----------- | ----------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Framework   | Astro 5 (`output: 'static'`)                    | Pure static, **no adapter** (unlike Next.js)                                                                                                                                                        |
| Content     | Content Layer API + `glob()` loader, Zod schema | Defined in root `content.config.ts`. Base dir is `./src/content/wiki` (subdirectory required to avoid Astro's legacy auto-collection of `src/content/<locale>/` folders).                           |
| MDX         | `@astrojs/mdx` ^4.3.x                           | **mdx 3.x fails with Astro 5.18** (`./jsx/renderer.js` not in exports). mdx 4.x pairs with astro 5.x; mdx 7.x needs astro 7.x.                                                                      |
| Styles      | Tailwind CSS 3 + `@astrojs/tailwind`            | Theme via CSS variables mapped in `tailwind.config.mjs` (shadcn-style tokens).                                                                                                                      |
| Icons       | `astro-icon` + `@iconify-json/lucide`           | Use `lucide:` prefix on every icon name. `reddit` does NOT exist in lucide (use `globe`).                                                                                                           |
| UI          | **Pure Astro native components (`.astro`)**     | Do NOT introduce React/Vue/Svelte runtime. Use `<details>`/`<dialog>` + minimal vanilla JS for interactivity.                                                                                       |
| i18n        | Astro built-in (`prefixDefaultLocale: false`)   | English has no `/en` prefix, others prefixed. Spread `[...locales]` into config — Astro's `Locales` type rejects readonly tuples.                                                                   |
| Sitemap     | `@astrojs/sitemap`                              | Auto-generates hreflang alternates from the i18n config.                                                                                                                                            |
| Deploy      | Cloudflare Pages                                | `pnpm build` → `dist/`                                                                                                                                                                              |
| Pkg manager | pnpm 11                                         | **`allowBuilds:` in `pnpm-workspace.yaml`** (NOT `onlyBuiltDependencies` — that's pnpm 10, dead in v11). esbuild + sharp need build approval or `astro build` fails during its pre-build dep check. |
| Node        | 20 LTS                                          |                                                                                                                                                                                                     |

## Architecture: Three-Layer Separation (critical)

This is the core design principle inherited from the course template. **Respect it in every edit:**

```
框架层 (src/pages, src/components, src/lib)      — fork-once, never edit per-game
配置层 (src/config, src/i18n/routing.ts, globals.css, public/) — edit once per game
内容层 (src/content, src/locales)                — fully replace per game
```

- Changing content must not touch framework code.
- Changing config must not rewrite framework.
- Framework layer should have **zero** game-specific strings.

## Hard Rules (from PRD — these are non-negotiable)

1. **All UI text comes from JSON** (`src/locales/<locale>.json`), never hardcoded in components.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PNGTRID/AnvilWiki](https://github.com/PNGTRID/AnvilWiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
