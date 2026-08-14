---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Layout

This is a **multi-stack design system monorepo**. The same design system is implemented in 4 stacks that share content, themes, and guidelines:

- `nortear-design-system-react/` — React 19 + `@base-ui/react` — port **6006**
- `nortear-design-system-vue/` — Vue 3 + `reka-ui` — port **6007**
- `nortear-design-system-svelte/` — Svelte 5 + `bits-ui` — port **6008**
- `nortear-design-system-vanilla/` — Vanilla TS factories + CSS `.nds-*` — port **6009**

Shared (read by all stacks):
- `docs/shared/content/<slug>/translations.json` — pt-BR/en/es content per component
- `docs/shared/guidelines/` — cross-stack rules
- `docs/shared/themes/` — CSS custom property themes
- `docs/shared/skill-refs/` — schemas/references consumed by `.claude/commands/*.md` skills
- `scripts/audit.mjs` and `scripts/audit-translation-literals.mjs` — deterministic checks

Per-stack guidelines live in `nortear-design-system-<stack>/guidelines/` and each stack has its own `CLAUDE.md`.

## Common Commands

Each stack is an independent npm package; run commands from inside the stack directory.

```bash
# Storybook (the primary developer interface — NOT App.tsx/main.ts)
npm run storybook          # React:6006 · Vue:6007 · Svelte:6008 · Vanilla:6009

# Build + typecheck
npm run build              # tsc -b && vite build (varies per stack)

# Lint
npm run lint

# Tests (React only has unit tests; all stacks have Storybook tests)
npm run test               # React: vitest run
npm run test:watch         # React: vitest watch
npm test                   # all stacks: Storybook Test (vitest browser) — play functions + axe via addon-a11y

# Visual regression
npm run chromatic
```

### Repo-root scripts

```bash
node scripts/audit.mjs <slug> --json              # quick deterministic audit per component
node scripts/audit.mjs --all --json
node scripts/audit-translation-literals.mjs       # audita o conteúdo compartilhado (5 seções)
node scripts/audit-translation-literals.mjs --only cobertura   # chaves *Code sem variante por stack
node scripts/audit-translation-literals.mjs --only plataforma  # texto preso a navegador (custo de portar)
node scripts/audit-translation-literals.mjs --only soltos      # snippet preso em override de stack
npm run core:pack                                # empacota docs/shared como @nortear/ds-core
```

## Architecture

### Storybook is the home

Each stack uses Storybook 10 as its primary interface. `App.tsx`/`main.ts` exists only as a development sandbox. New components are added by creating `*.stories.tsx` and `*Docs.tsx` (or stack equivalent) — **not** by registering in `App.tsx`.

The Storybook sidebar order is controlled by `storySort` in `.storybook/preview.ts`. Brand themes are toggled via toolbar globals.

### Component anatomy (per stack)

For a component `<slug>`, each stack has:

- `src/components/ui/<slug>*.{tsx,vue,svelte,ts}` — primitive
- `src/components/ui/<slug>*.stories.*` — Playground + variant/state/composition stories
- `src/components/docs/<Slug>Docs.*` — full docs page (consumed via `withAutoDocsTab` HOC)
- `src/components/docs/shared/sections/Docs*.*` — 15 generic section containers used by every docs page (header, anatomy, when-to-use, do-dont, import, variants, states, props, tokens, accessibility, related, notes, analytics, testes, demonstration)

Content for all of those comes from `docs/shared/content/<slug>/translations.json`. Code snippets in the JSON (keys ending in `Code`) carry one variant per stack; descriptive text must be API-neutral.

### Shared `lib` per stack

Each stack has these files in `src/lib/`:

- `i18n.ts` — `useTranslation(translations, overrides?)` with dot-path lookup and per-stack overrides via the `overrides` parameter (use for stack-specific prop names that differ from the shared JSON)
- `use-seo.ts` — `useSeoEffect` / `applySeo`. Detects iframe context and writes meta tags into the parent (Storybook manager). Title is `${title} · Design System` — **do not include the suffix in `seo.title` in translations.json**.
- `use-active-section.{ts,svelte.ts}` — IntersectionObserver wrapper. `onActive` (highlight) fires immediately; `onDwell` (analytics `docs_section_viewed`) fires only after 2s continuous visibility, suppressing false positives during programmatic scroll from nav clicks.
- `analytics.ts` — `track(event, params)`. GA4 lives in `manager-head.html` (not the iframe) with `send_page_view:false`; `track()` calls `window.top.gtag`.
- Sanitização: qualquer `dangerouslySetInnerHTML` / `v-html` / `{@html}` / `innerHTML` de conteúdo dinâmico usa `DOMPurify.sanitize()` **direto no call site** (`import DOMPurify from 'dompurify'`) — sem wrapper local, para que SAST reconheça o sanitizador (ver guideline 09).

### Cross-stack translation strategy

Different UI libraries (`@base-ui` / `reka-ui` / `bits-ui` / factories) expose different prop names for the same concept. The convention is:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [julianamucci/design-system](https://github.com/julianamucci/design-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
