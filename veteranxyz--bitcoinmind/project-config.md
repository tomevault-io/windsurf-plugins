---
trigger: always_on
description: This file defines how AI coding agents and automated editing tools should work in this repository.
---

# BitcoinMind Agent Guide

This file defines how AI coding agents and automated editing tools should work in this repository.

BitcoinMind is an Astro-based editorial study site for Bitcoin as money, protocol, custody practice, and sovereignty. The project depends on stable information architecture, curated content, restrained visual design, and static-first implementation.

Before changing UI, layout, copy, routes, or content structure, read:

```text
DESIGN.md
```

## 1. Operating Model

Think of the repository in four layers:

```text
Content layer        src/data/**, selected src/pages/** copy
Design layer         DESIGN.md, src/styles/design-system.css, src/styles/styles.css
Application layer    Astro pages, components, Preact islands, browser scripts
Deployment layer     package.json, astro.config.mjs, wrangler.jsonc, worker/index.js, public/_headers, CI
```

Make changes in the correct layer.

Do not solve:

- a content problem with global CSS
- a token problem with one-off component overrides
- a route problem by rewriting unrelated pages
- a local UI issue by replacing the whole design system
- a documentation mismatch by ignoring the actual codebase

## 2. Current Stack

The current stack is defined by `package.json`. At this snapshot, the project uses:

```text
Astro 7
@astrojs/preact 6
Preact 10 islands
TypeScript 5
CSS custom properties
Fontsource: Literata, Inter, Geist Mono
GitHub Actions CI
Cloudflare Workers static assets via Wrangler with a small Worker routing script
Node.js 22.12 or newer for local and CI builds
```

Important files:

```text
package.json
astro.config.mjs
wrangler.jsonc
worker/index.js
public/_headers
src/layouts/Base.astro
src/styles/design-system.css
src/styles/styles.css
src/components/Nav.astro
src/components/*.astro
src/components/*.tsx
src/data/*.ts
src/data/routes.json
src/data/site.json
src/lib/seo.ts
src/lib/routes.ts
src/pages/**/*.astro
src/pages/sitemap.xml.ts
src/scripts/*.ts
scripts-build/*.mjs
```

If this section disagrees with `package.json`, `astro.config.mjs`, or `wrangler.jsonc`, trust the codebase and update this file.

## 3. Commands

Install dependencies:

```bash
npm ci
```

Run local development:

```bash
npm run dev
```

Run validation:

```bash
npm run validate
```

Run individual validation stages when diagnosing a failure:

```bash
npm run check
npm run build
npm run audit
npm run test:browser
```

Refresh generated data and assets:

```bash
npm run refresh-data
```

Preview production output:

```bash
npm run preview
```

Before reporting completion for source-code changes, run:

```bash
npm run validate
```

If a command cannot be run, state exactly which command was not run and why.

## 4. Task Types

Classify the task before editing.

### Documentation-Only Task

Likely files:

```text
README.md
DESIGN.md
AGENTS.md
```

Rules:

- Do not modify source code.
- Keep technical claims aligned with current files.
- Avoid turning project documentation into promotional copy.
- If describing routes or dependencies, verify them against the repository.

### Design-System Task

Likely files:

```text
src/styles/design-system.css
src/styles/styles.css
src/layouts/Base.astro
src/components/*.astro
```

Rules:

- Use existing tokens first.
- Add tokens only for recurring roles.
- Do not hardcode colors in components.
- Preserve Literata / Inter / Geist Mono unless changing the font system is the explicit task.
- Preserve the quiet editorial identity.

### Content or Curation Task

Likely files:

```text
src/data/*.ts
src/pages/notes.astro
src/pages/about.astro
```

Rules:

- Preserve TypeScript shapes.
- Do not reorder curated lists unless the task asks for sequencing work.
- Do not bulk-generate filler resources.
- Keep tone precise, non-hype, and source-aware.
- Prefer adding content to datasets rather than duplicating it inside pages.

### Information Architecture Task

Likely files:

```text
src/pages/**
src/components/Nav.astro
src/lib/seo.ts
src/data/routes.json
src/pages/robots.txt.ts
```

Rules:

- Treat route changes as high risk.
- Preserve canonical metadata unless intentionally changing it.
- Update navigation, page metadata, sitemap behavior, and internal links together.
- Do not rename routes as an opportunistic cleanup.

### Interactive Behavior Task

Likely files:

```text
src/components/*.tsx
src/scripts/*.ts
src/pages/frames/*.astro
scripts-build/*.mjs
public/pulse.json
```

Rules:

- Keep islands small and purposeful.
- Do not convert static pages into a broad client-side app.
- Preserve accessible labels, keyboard behavior, and mobile behavior.
- Keep Frames educational rather than trading-oriented.
- Handle unavailable live data with fallbacks.

### Deployment or Build Task

Likely files:

```text
package.json
package-lock.json
astro.config.mjs
wrangler.jsonc
public/_headers
.github/workflows/*
scripts-build/*.mjs
```

Rules:

- Avoid changing dependency versions casually.
- Keep CI commands aligned with `package.json`.
- Preserve Cloudflare static-assets behavior unless deployment strategy changes.
- Do not remove defensive fallback behavior from data-refresh scripts.

## 5. Route Constraints

The current public routes are defined by `src/pages/**`. At this snapshot they include:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VeteranXYZ/BitcoinMind](https://github.com/VeteranXYZ/BitcoinMind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
