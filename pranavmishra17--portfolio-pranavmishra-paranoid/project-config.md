---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo layout

The entire app lives in `portfolio/` (Create React App). The repo root only holds README/LICENSE/preview assets — all commands run from inside `portfolio/`.

```
portfolio/
  src/
    App.js                    # Router only: "/" → MainPortfolio, "/resume" → ResumeViewer
    MainPortfolio.js          # The single-page layout — wires every section together
    data/                     # ALL editable content lives here (see below)
    components/               # All UI pieces
    styles/About.css          # About-section styles (one-off)
  public/
    assets/images/{ai_ml,game_design,misc,companies,achievements,icons,default}/
    resumes/{ai,game}/        # Drop PDFs here; resumeUtils.js probes common filenames
```

## Commands

Run everything from `portfolio/`:

- `npm start` — dev server on http://localhost:3000 (prestart auto-runs `resume-manifest`)
- `npm run build` — production build (uses `CI=false` so warnings don't fail the build; bash-only — on Windows CMD use `set CI=false && npx react-scripts build`)
- `npm run resume-manifest` — scan `public/resumes/{ai,game}/` for PDFs and write `manifest.json`; auto-runs via `prestart` / `prebuild`
- `npm test` — Jest via react-scripts
- `npm run check` — `npm install && npm run build` (CI sanity check used in RUN_LOCAL.md)

Stack: React 19, react-router-dom 7, framer-motion 12, react-scripts 5, plain CSS (no Tailwind, no TS).

## Where each piece of content lives — the only files you usually edit

All site content is data-driven from three files in [portfolio/src/data/](portfolio/src/data/). You almost never need to touch components to add content.

### Projects — [portfolio/src/data/projects.js](portfolio/src/data/projects.js)

One `projects` object with three arrays, rendered in this order on the page: `aiMl`, then research (publications), then `gameDesign`, then `misc`. Each project shape:

```js
{
  id: "unique-kebab-id",
  title, category, description,
  mainImage: "assets/images/<bucket>/foo.png",   // path RELATIVE to public/, no leading slash
  gallery: ["assets/images/<bucket>/...", ...],  // [] is fine
  techStack: ["..."],
  githubLink, demoLink, websiteLink              // empty string "" hides the button
}
```

To **add a project**: drop the image into `public/assets/images/{ai_ml|game_design|misc}/`, append an object to the right array. To **remove**: delete the entry. `getImageWithFallback` at the bottom of the file handles missing/empty image paths by substituting a category default from `public/assets/images/default/`.

Also in this file: `contactInfo` (name, title, bio, emails, social URLs, resume paths) and `socialIcons` (icon image refs).

### Experience — [portfolio/src/data/experience.js](portfolio/src/data/experience.js)

Flat `experiences` array. Rendered in [ExperienceModal.js](portfolio/src/components/ExperienceModal.js), opened by the "Experience" tab in [CollapsibleSectionTabs.js](portfolio/src/components/CollapsibleSectionTabs.js) (see `onExperienceToggle` wiring in [MainPortfolio.js:291](portfolio/src/MainPortfolio.js#L291)). Each entry has `company`, `title`, `duration`, `location`, `companyLogo` (path under `public/assets/images/companies/`), `description` (bullet array), `techStack`, `links`.

### Publications / Research — [portfolio/src/data/publications.js](portfolio/src/data/publications.js)

`publications` array + a derived `publicationStats` object. Rendered by [PublicationsSection.js](portfolio/src/components/PublicationsSection.js) under section id `research`. Each entry has full abstract, `status` ("ACCEPTED" | "Under Review" | "Under Preparation"), `doi`, `pdfLink`, `codeLink`, `citationCount`, `techStack`, `tags`, `keywords`. Stats (total publications, total citations) auto-recompute from the array — no manual counter to update.

## Page architecture — what renders what

[MainPortfolio.js](portfolio/src/MainPortfolio.js) is the whole page. It owns three pieces of state worth knowing:

- `activeSection` — driven by a scroll listener that registers each `<section>`'s offset via `sectionsRef`; controls nav highlight.
- `connectSlateOpen` — toggled by the **ConnectButton** floating next to the profile image and by every "Connect" CTA. Renders [ConnectSlate.js](portfolio/src/components/ConnectSlate.js) inline beneath the profile pic. The hardcoded social URLs (GitHub/LinkedIn/Resume/YouTube) live in the `BUTTONS` array at the top of that file — NOT in `contactInfo`.
- `experienceModalOpen` — opened from the collapsible Experience tab, renders [ExperienceModal.js](portfolio/src/components/ExperienceModal.js).

Section order in [MainPortfolio.js](portfolio/src/MainPortfolio.js): About → CollapsibleSectionTabs (Experience/Publications launchers) → ProjectTabs (sticky nav) → AI/ML → Research (publications) → Game Design → Misc → MinimalContactBelt.

## The visual gimmicks — where they live


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PranavMishra17/Portfolio-PranavMishra-Paranoid](https://github.com/PranavMishra17/Portfolio-PranavMishra-Paranoid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
