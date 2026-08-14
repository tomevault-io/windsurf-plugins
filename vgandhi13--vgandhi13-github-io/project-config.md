---
trigger: always_on
description: Personal academic website for Varun Gandhi (MS CS @ UMass Amherst, AI research / LLM post-training).
---

# CLAUDE.md

Personal academic website for Varun Gandhi (MS CS @ UMass Amherst, AI research / LLM post-training).
Static Astro site. Design goal: **minimal and easy to maintain** — resist adding dependencies,
frameworks, or client-side JS beyond what exists.

## Commands

```sh
npm run dev      # dev server at http://localhost:4321 (hot reload)
npm run build    # static build into dist/ — ALWAYS run to verify changes
npm run preview  # serve dist/
```

No tests. Verification = clean build + grep the built HTML in `dist/` for expected output.
After making changes, if `npm run dev` isn't already running, start it in the background by
default so the user can manually look at the result in a browser.

**Deploy**: live at https://vgandhi13.github.io — pushing to `main` on
github.com/vgandhi13/vgandhi13.github.io triggers `.github/workflows/deploy.yml`
(withastro/action → GitHub Pages).

## Layout of the repo

- `src/pages/index.astro` — the entire homepage: hero (profile pic, bio, social icon links),
  jump-nav (News · Research · Teaching), Karpathy-style experience **timeline**, scrollable
  **News** box, Research (papers), Teaching. All homepage edits happen here.
- `src/layouts/Base.astro` — header/footer, theme toggle, **all global CSS + color variables**.
- `src/layouts/Entry.astro` — wrapper for a single note/blog post (imports KaTeX CSS).
- `src/content/notes/*.md`, `src/content/blog/*.md` — content collections (schema in
  `src/content.config.ts`: `title`, `description?`, `date`, `updated?`, `draft`). Filename =
  URL slug. `date` = created (immutable); bump `updated` to today whenever revising content —
  the entry page shows "· Updated <date>" when it differs from `date`.
- `src/components/Search.astro` + `src/pages/search-index.json.ts` + `src/lib/stopwords.js` —
  client-side full-text search (see below).
- `public/profile.jpg`, `public/logos/*.png`, `public/images/notes/*` — static assets.
- `Notes/` and `Context/` — **the user's raw source material, NOT built into the site.**
  One folder per topic: `Notes/<Topic>/main.md` with source images alongside it (e.g.
  `Notes/InstructionFinetuning/main.md` + `lr1image.png`). These rough drafts get converted
  into `src/content/notes/` (see the publish-content skill). `Context/*.txt` holds bio/CV
  facts used to fill homepage sections.

## Conventions & gotchas

- **After adding any new feature or convention**, check whether it should be recorded for
  future sessions: update an existing skill (`.claude/skills/`), create a new one if it's a
  repeatable workflow, or add a line here — so the next agent can pick up where this one left
  off. Prefer one home per fact (skill for authoring workflows, CLAUDE.md for site-wide
  facts/gotchas) to avoid drift.

- **Never use em dashes (—)** in notes, blog posts, or any prose on this site. Use a comma,
  colon, semicolon, or parentheses instead, whichever fits the sentence. En dashes (–) in
  numeric ranges (e.g. `5–8 × 10^-5`) or compound modifiers are fine and not affected by this rule.
- **Stale Vite dep cache**: if client-side JS (e.g. search) silently stops working in dev and
  the console shows `504 Outdated Optimize Dep`, kill the dev server, `rm -rf
  node_modules/.vite`, and restart. Happens when deps change while the server runs.
- **Astro scoped styles don't reach JS-created DOM.** Any element built with
  `document.createElement` needs its styles in a `<style is:global>` block (this bit us in
  Search.astro). Keep such selectors namespaced (`.search-*`).
- **Theme**: light/dark via `data-theme` on `<html>`, set pre-paint by an inline script in
  Base.astro, persisted in localStorage, falls back to `prefers-color-scheme`. All colors are
  CSS variables in Base.astro (`--bg, --text, --heading, --text-muted, --link, --border,
  --surface, --mark-bg`). Dark palette was contrast-tuned (WCAG AA, body ~10:1, muted ~5:1);
  keep headings brighter than body text.
- **Search**: `/search-index.json` is generated at build from both collections. Each doc has
  `text` (stop-word-stripped, for Fuse.js fuzzy matching), `plain` (readable, for sentence
  previews), and `description` (frontmatter, shown when only the title matches). Stop words
  live in `src/lib/stopwords.js`, shared by index and query side. Result cards: query words
  are highlighted in the title whenever they match; preview priority is body sentence
  (deep-linked via `#:~:text=`) → description → first 140 chars. `<Search scope="note|blog">`
  sets the placeholder and ranks that collection first (other collection still shown, badged).
  Stay dependency-light: Fuse.js only.
- **Search ranking is scope → literal overlap → Fuse score**, in that order (`Array.sort` is
  stable, so Fuse's own order, including its 2× title weight, breaks ties). "Literal overlap"
  = for each query token, the length of its longest prefix literally present in title+`plain`,
  **summed across tokens** so multi-token queries rank on total coverage. Summing matters: on
  "proximal policy optimization" both RL notes tie at 12 on the longest single token
  ("optimization"), and only the sum (26 vs 25) separates a full `proximal` from `proxima`
  found inside "ap**proxima**tion".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vgandhi13/vgandhi13.github.io](https://github.com/vgandhi13/vgandhi13.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
