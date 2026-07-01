---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Personal style — defer to lailai.skill

lailai's **general, cross-project personal style** lives in the **lailai.skill** submodule at [`.claude/skills/lailai-skill/`](skills/lailai-skill/SKILL.md). Read its `SKILL.md`, then the relevant `references/` for any task touching personal voice, writing, code, or design:

- Chinese writing voice, wording (你/仅/若, no 显然/易得), Markdown, LaTeX math (`$...$`), AI-tone blacklist → `references/{writing-style,wording,markdown-style,latex-math-style,ai-tone-blacklist}.md`
- OI C++ style, engineering-code comments, design principles (统一·简约·现代), project/README/commit conventions → `references/{cpp-oi-style,engineering-code-style,design-style,project-docs-style}.md`
- Who lailai is, how he thinks/decides → `profile/`

**This repo's `.claude/` holds only project-specific config** — site architecture, commands, deploy, and the path-scoped `rules/` for _this_ Docusaurus site (i18n, laikit components, site-authoring conventions). It does **not** duplicate the general rules; those live in the skill. When a `rules/*.md` covers only the site-specific slice, it points to the skill for the general part.

After cloning, init the submodule: `git submodule update --init`. Update it later with `git submodule update --remote .claude/skills/lailai-skill`.

## Project

Source for [lailai's personal website](https://lailai.one), built with Docusaurus 3 (TypeScript). Deployed via GitHub Actions to GitHub Pages and rsynced to a custom server (see `.github/workflows/deploy.yml`). Requires Node `>=20` (CI uses Node 24).

## Commands

```bash
npm start                 # Dev server (default English locale)
npm run start:zh-Hans     # Dev server in Simplified Chinese
npm run build             # Production build into ./build
npm run serve             # Serve the built site locally
npm run clear             # Clear Docusaurus cache (.docusaurus)

npm run i18n              # Regenerate translation JSON for zh-Hans
npm run format            # Prettier write across the repo
npm run typecheck         # tsc, no emit (uses tsconfig.json)
npm run check             # i18n + format + typecheck — run before every commit
```

There is no test runner. `npm run check` is the gate.

## Architecture

### Content vs. code

- `docs/` — three doc sets (`contest/`, `note/`, `project/`) wired up in `sidebars.ts` as `contestSidebar` / `noteSidebar` / `projectSidebar`. Sidebars are hand-curated, not auto-generated.
- `blog/` — MDX posts grouped by topic folders; `authors.yml` and `tags.yml` define the controlled vocabularies (`onInlineTags`/`onInlineAuthors` warn if a post uses an unlisted value).
- `src/pages/` — custom React pages (`about`, `travel`, `friends`, `resources`, `settings`, `insights`, `changelog`, `privacy`, plus the bespoke `index.tsx` cover/home). `insights` is live Umami traffic data. Two custom pages live under the blog area and reuse `BlogShared/Scaffold` (blog chrome + sidebar): `src/pages/blog/moments` and `src/pages/blog/overview`. `blog/overview` is build-time blog-content stats (reads `src/utils/blogData`; KPI `DataCard`s + two laikit `Chart`s — a bar and a line) and renders as the **first** tab of `BlogShared/ArchiveTabs` (`ArchiveTabsNav` — Overview / By Year / By Tag / By Author); the sidebar "Archive" link lands on it (`/blog/overview`). Add new archive tabs there. Page-local React lives in a `_components/` subfolder beside the page (e.g. `insights/_components`, `travel/_components`, `blog/overview/_components`) — the leading `_` stops Docusaurus from routing it.
- `src/theme/` — swizzled Docusaurus theme overrides (Layout, Blog\*, MDXComponents, Admonition, Root). Use `npm run swizzle` rather than copying files by hand.
- `src/components/laikit/` — the in-house design system (`Card`, `TitleCard`, `LinkCard`, `DataCard`, `Chart`, `Badge`, `Button`, `Segmented`, `Slider`, `Switch`, `Tooltip`, `Skeleton`, `IconBlock`, `Page`, `Section`, etc.). `Chart` is the one shared time-series chart (bar/line, rounded gridlines + hover crosshair/Tooltip + optional `loading` skeleton); both `blog/overview` and `insights` (Pageviews) render it — callers pass `ChartDatum[]` with pre-formatted `tooltipLabel`/`axisLabel`, so the component stays free of any date/locale logic. Reuse these primitives before creating new components. The full design-system ruleset — primitive inventory, per-component-folder layout, hover-motion limits, the CSS-Module silent-failure caveat — lives in `.claude/rules/components.md`, auto-loaded whenever files under `src/**` are touched.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lailai0916/lailai0916.github.io](https://github.com/lailai0916/lailai0916.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
