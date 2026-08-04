---
trigger: always_on
description: Project orientation for AI agents working on this repository. If you're a
---

# CLAUDE.md — agentlanguages.dev

Project orientation for AI agents working on this repository. If you're a
human contributor, [README.md](README.md) is the front door and
[CONTRIBUTING.md](CONTRIBUTING.md) covers PR mechanics; you may still find
this document useful as a map of how things fit together.

---

## What this repo is

A community-edited catalogue of programming languages designed for AI agents
to *author* code. It is **not** a Vera marketing site (`veralang.dev`) or a
Negroni Venture Studios product. It is a neutral directory that catalogues
Vera alongside ~30 other languages with the same metadata format, organised
around three philosophical camps (Syntactic, Verification, Orchestration)
plus Adjacent and Unclassified.

The taxonomy and editorial frame come from
["Three camps alike in dignity"](https://negroniventurestudios.com/2026/05/20/three-camps-alike-in-dignity/)
(Negroni Venture Studios blog, May 2026). The catalogue is descriptive, not
promotional; it does not rank entries.

Live at [agentlanguages.dev](https://agentlanguages.dev). MIT for code,
CC BY 4.0 for content.

---

## Architecture in one screen

- **Framework:** Astro 6.x, static-output. No JS framework. Vanilla JS only
  for the theme toggle.
- **Content:** A single content collection at `src/content/languages/*.md`,
  schema-validated by Zod in `src/content.config.ts`. Each MDX file is one
  language entry; frontmatter is required, body is optional.
- **Pages:**
  - `src/pages/index.astro` — homepage
  - `src/pages/languages/[slug].astro` — detail pages (only rendered for
    entries with a non-empty body)
  - `src/pages/languages/[slug].md.ts` — markdown companion per entry
  - `src/pages/{index.md,llms,llms-full,robots,sitemap}.ts` — agent-
    accessibility endpoints (see below)
  - `src/pages/.well-known/ai-plugin.json.ts` — agent plugin manifest
- **Styling:** One global stylesheet at `src/styles/global.css`. CSS custom
  properties for tokens; CSS `@media (prefers-color-scheme: dark)` for OS
  dark-mode detection, with `[data-theme]` attribute overrides for the
  user-facing toggle.
- **Shared editorial prose:** `src/data/homepage-copy.ts` exports the
  Three Camps narrative and methodology copy as a single source of truth;
  both the HTML page and the markdown companion import it.
- **Star counts:** Baked in at build time from `src/data/stars.json`,
  refreshed weekly by `.github/workflows/refresh-stars.yml`. **No** live
  GitHub API calls from the site.
- **Deploy:** GitHub Pages, served from the `gh-pages` branch. The
  `.github/workflows/deploy.yml` workflow runs on every push to `main` and
  also via `workflow_run` after the stars refresh completes (so weekly
  refreshes propagate without a separate human-driven deploy).

---

## Agent-accessibility surface

Every editorial surface has a machine-readable companion served on the same
origin. The pattern is modelled on
[veralang.dev/docs](https://github.com/aallan/vera/tree/main/docs) and the
[llmstxt.org](https://llmstxt.org) conventions.

- **`/robots.txt`** — All agents welcome. Points at `/sitemap.xml`.
- **`/llms.txt`** — Short index following the llmstxt.org spec. Grouped by
  camp; each entry has its one-liner and a link to its `.md` companion.
- **`/llms-full.txt`** — Every entry concatenated into one file (~150 KB).
  An agent that wants the entire catalogue can fetch it in one round-trip
  rather than once per entry.
- **`/index.md`** — Markdown companion to the homepage (Three Camps
  narrative + catalogue index + methodology).
- **`/languages/<slug>.md`** — Markdown companion for every detail page.
- **`/sitemap.xml`** — Hand-rolled (in `src/pages/sitemap.xml.ts`) because
  `@astrojs/sitemap` ignored non-HTML routes and would have hidden every
  markdown surface from crawlers.
- **`/.well-known/ai-plugin.json`** — Agent-plugin manifest with a dynamic
  count of catalogued projects in `description_for_model`.

Every HTML page advertises its companion via `<link rel="alternate"
type="text/markdown">`, and the homepage's methodology section has a
visible "For machines" column pointing at the same surfaces. Detail pages
expose the alternate in their `metaprops` block ("markdown: vera.md").

---

## Editorial conventions

Get these right and the maintainer's review pass is much faster.

- **British English in prose.** "Licence" (noun), "licensed" (verb),
  "catalogue", "organisation", "behaviour", "favour", "specialised". The
  Zod schema's field is named `license` (US) — that's a schema name, not
  prose, leave it alone.
- **No sentences ending in "is".** Reword: "what the problem is" →
  "how to frame the problem" / "the shape of the problem itself" /
  "what the problem amounts to." The pattern shows up surprisingly often;
  catch it on re-read.
- **"Distinctive move" not "provocative move."** Sitewide convention. The
  word "provocative" is reserved for the rare case where a project is
  taking a deliberately contrarian stance (Prove's anti-AI-training licence
  is the canonical example).
- **Descriptive, not promotional.** Verbs that locate authority in the
  project: "ships", "claims", "reports", "publishes", "states". Avoid:
  "elegant", "powerful", "next-generation", "best", "of course",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aallan/agentlanguages](https://github.com/aallan/agentlanguages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
