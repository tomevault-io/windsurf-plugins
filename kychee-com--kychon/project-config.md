---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Kychon is an AI-powered membership/community portal template built on the Run402 serverless platform. It's three products in one:

- **Kychon** - A forkable community portal template (free + $5-20/mo Run402 hosting)
- **Kychon Studio** - AI concierge that builds your portal via investigation + interview ($29 premium)
- **Kychon Pro** - Ongoing AI customization agent ($9-29/mo)

**Status**: Design complete, ready for implementation. The full spec lives in `docs/spec.md`.

**Cross-repo boundary**: The marketing site at `kychon.com` lives in the sibling private repo `kychee-com/kychon-private`. Marketing-site source, deploy script, copy, and domain config changes go there, not here. See `openspec/specs/marketing-deploy/spec.md`.

## Git workflow — worktrees, never branches

Multiple agent sessions work these repos in parallel. Isolation is by **git
worktree**, not by branching or stashing.

- **ALWAYS worktree.** Do work inside a dedicated worktree (`git worktree add`).
  Never edit, branch-switch, or otherwise disturb a shared checkout — another
  session may be using it, and switching branches under them destroys their work.
- **NEVER create branches.** No feature-branch or PR ceremony on your own
  initiative. A worktree's auto-created backing branch is plumbing, not workflow.
- **NEVER `git stash`.** It is invisible to everyone else and easy to lose.
- **Small changes commit directly to `main`, no ceremony.** From a worktree,
  push with `git push origin HEAD:main` (expect to rebase).
- If you find yourself on a branch in a primary checkout, you are already in the
  wrong place — make a worktree and clean up after yourself.

## Architecture

### Config-Driven Design

The central design principle: **an AI agent's API is SQL for config and file editing for code.** Three customization tiers:

1. **SQL only** (80%) - rebrand, toggle features, restructure via `site_config` table (JSONB)
2. **HTML/CSS** (15%) - visual/layout changes
3. **Full fork** (5%) - new tables, functions, page types

### Inline Editing ("The page IS the admin")

Members and admins see the same URL. Admins get edit overlays via `data-editable` attributes. Three editing layers:
- **Simple text**: native `contenteditable` (~30 lines JS)
- **Rich text**: Tiptap (~45kB), lazy-loaded only for admins
- **Images**: click-to-upload handler (~30 lines)

Member page load: ~15kB. Admin adds: ~60kB (admin-editor.js + Tiptap).

### Composable Layout (every block is data)

Every visible block on every page — including chrome (`zone='header'`, `zone='footer'`) and main content (`zone='main'`) — is a row in the `sections` table addressed by `(page_slug, zone, scope, position)`. There are no hard-coded `<Nav>` / `<Footer>` components.

- **Block registry** (`src/lib/blocks.ts`) — single isomorphic `renderBlock(section, ctx): string` runs at Astro build time (Node) and at runtime (browser). Dynamic blocks emit a skeleton at bake time and have a `hydrate(el, section, ctx)` that fetches data and replaces the body. Block types:
  - **Main-zone**: `hero`, `features`, `cta`, `stats`, `testimonials`, `faq`, `polls` (dyn), `event_countdown` (dyn), `announcements_feed` (dyn), `activity_feed` (dyn), `tagline_strip`, `promo_cards`, `link_list` (dyn in resources mode), `events_list` (dyn), `slideshow` (dyn), `custom`.
  - **Header-zone**: `nav`, `brand_header`, `sign_in_bar`, `page_banner` (page-scoped per-page banner; substrate uses `scope='page'` so a banner appears only on its page).
  - **Footer-zone**: `footer_address`, `footer_links`, `footer_copyright`, `footer_social`, `footer_attribution`.
  - The `slideshow` controller lives at `src/lib/blocks/slideshow.ts` (~3 kB minified) and respects `prefers-reduced-motion`, hover, focus, and tab-visibility pause; cleans up on `astro:before-swap` and `wl-content-rendered`.
- **Typed seeds** — each forkable project has a `src/seeds/{project}.ts` module exporting a `ProjectSeed`. `scripts/generate-seed-sql.ts` translates the typed seed into idempotent `seed.sql` (gitignored). `KYCHON_PROJECT` env var selects which project's seed to use; `Portal.astro` and the generator both read it via `getActiveProjectSeed()`.
- **Build-time bake** — `Portal.astro`'s frontmatter calls `renderBlock` against the active project's `scope='global'` header / footer blocks at build time (page-scoped chrome like `page_banner` is intentionally excluded so it does not leak into every page's static HTML — runtime hydrate paints it on its target page in the first frame). The result is injected into `#zone-header` and `#zone-footer` containers via `set:html`, giving instant chrome on cold visits with no flicker.
- **Runtime hydrate** — `src/lib/page-render.ts:hydratePage(slug)` runs on every page load. It reads cached sections from `localStorage` (`wl_cache_sections_{slug}`), renders into zones, then fetches fresh from PostgREST in one query (`?or=(and(page_slug.eq.{slug},scope.eq.page),scope.eq.global,page_slug.eq.*)`), updates if different, and runs each block's hydrator.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kychee-com/kychon](https://github.com/kychee-com/kychon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
