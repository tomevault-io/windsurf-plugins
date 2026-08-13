---
trigger: always_on
description: Guidance for Claude Code (and any AI assistant) working in this repo.
---

# CLAUDE.md

Guidance for Claude Code (and any AI assistant) working in this repo.

## What this is
Personal portfolio for **Stepan Roze** — live at **roze.live**. A single-page
React app. Two audiences via a **view mode** toggle: **Client** (hire me:
services, prices, projects) and **Company** (full CV: experience + skills).
Multilingual (en, nl, ar, es), dark/light theme.

## Who this site is for (2026-08 repositioning — read before changing copy)
The goal changed from winning freelance clients to **getting hired as an
employee**, at companies in Belgium, the Netherlands and the rest of Europe.
That decision drives most of what follows:

- **CV mode is the default view.** A recruiter seeing an hourly rate before the
  experience reads the page as a freelance pitch, not a job application.
- **Ukrainian was removed from the codebase entirely**, not hidden — the
  Ukrainian market is not a target and the strings were dead weight in every
  bundle. `L()` now takes `(en, nl, ar, es)` and `Language` has no `"uk"`, so
  new Ukrainian copy will not compile.
- **Dutch is excluded from browser auto-detection** even though the version
  exists. The audience has Dutch browsers, the author cannot proof-read Dutch,
  and weak Dutch costs more trust than plain English does in Benelux IT. Put it
  back only after a native speaker reviews the copy.
- **Client-only content must not leak into CV mode.** It reads as "looking for
  gigs" to someone considering an employment offer.

Client mode still exists and still works — it is one click away, not deleted.

## Stack
- **React 19 + Vite 8 + TypeScript 5.9**
- **Tailwind CSS v4.3**, **Motion** (`motion/react`, the framer-motion successor)
- Services carousel is **CSS scroll-snap**, not a library (react-slick is gone)
- **Supabase** edge function backend (`make-server-a62f57c7`), now used for the
  contact form only (three plain `fetch` callers: `contact-section`,
  `book-call-fixed`, `scroll-to-top-button`)
- Deploy: **Netlify** (`netlify.toml`). `dist/` is **committed to git**.

**Vite 8 bundles with rolldown**, so build config is `build.rolldownOptions`
(not `rollupOptions`) and chunking uses `output.advancedChunks.groups`.
`esbuild` is an explicit devDependency only because `@tailwindcss/node` still
imports it and pnpm does not hoist it.

## Commands
- Dev server: `npm run dev` → **http://localhost:3000/** (another project may
  already hold that port — pass `--port` if the page looks like someone else's)
- `npm run check` = typecheck + lint + build. **All three must be green**;
  they are as of the 2026-07 cleanup, so any error you see is yours.
- Lint is at **0 errors / 20 warnings**, and `--max-warnings 20` is a ratchet:
  the count may go down, never up (it went 32 → 20 when the account surface was
  removed). The remainder needs a judgement call at each call site, so do not
  silence them in bulk.
- Netlify build command: `npm run build` → publishes `dist/`
- **Package manager is pnpm** (`pnpm-lock.yaml`); install via
  `corepack pnpm install`. pnpm 11 no longer reads the `pnpm` field from
  package.json — **overrides and build-script approvals live in
  `pnpm-workspace.yaml`**. `@types/react` is pinned there: transitive peers pull
  their own copy, and two copies of the React types make every component
  "not a valid JSX element type".
- **Do not push unless asked.** Commit when asked; the user often reviews first.

## Where things live
- `src/app/App.tsx` — providers + top-level page routing (hash based)
- `src/app/components/main-page.tsx` — renders sections per view mode
- `src/app/components/hero-ultra-modern.tsx` — hero (client = agency split,
  company = professional profile)
- `src/app/components/services-creative-slider.tsx` — services + **prices**
  (hardcoded array, NOT from translations)
- `src/app/components/how-i-work.tsx` — process + honest part-time timelines
  (self-contained translations via an inline `L()` helper). **Client mode only**
  — its content ("Intro — free", "Quote & timeline") is a sales funnel, and it
  was 1 528 px of it sitting in the middle of the CV.
- `src/app/components/view-mode-toggle.tsx` — the Client/CV switch, rendered
  **inline inside the header** by `navigation.tsx`. It used to be three
  fixed-position variants that all covered page content; do not float it again.
- `src/app/components/portfolio-creative-slider.tsx` — projects grid (owned work
  only: marinek.store, roze.live) → opens `project-fullscreen-view.tsx`
- `src/app/components/github-showcase.tsx` — live GitHub feed (@irozedev)
- `src/utils/translations.ts` — **huge** 4-language dictionary. Newer components
  skip it and localize inline with an `L(en,nl,ar,es)` helper.
- `src/app/contexts/language-context.tsx` — `OFFERED` is the single source of
  truth for which languages exist. The switcher, `?lang=`, the saved choice and
  auto-detection all validate against it; a language removed there is also
  rejected out of a visitor's stale `localStorage`.
- `src/app/contexts/view-mode-context.tsx` — `client` | `cv`, `setViewMode`.
  **Defaults to `cv`.** The saved choice is read in the `useState` initialiser,
  not an effect (as an effect it painted the wrong mode for one frame), and the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [irozedev/my-portfolio](https://github.com/irozedev/my-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
