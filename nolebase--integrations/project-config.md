---
trigger: always_on
description: Concise but detailed reference for contributors working across the `nolebase/integrations` monorepo. Improve code when you touch it; avoid one-off patterns.
---

# Nolebase Integrations Agent Guide

Concise but detailed reference for contributors working across the `nolebase/integrations` monorepo. Improve code when you touch it; avoid one-off patterns.

## Tech Stack (by surface)

- **Docs app (`docs/`)**: VitePress 2 (alpha), Vue 3, UnoCSS, TypeScript, Vite.
- **Packages (`packages/*`)**: TypeScript ESM libraries, mostly VitePress plugins, Markdown-It plugins, and Vue UI components.
- **Tooling**: pnpm workspace, Vitest, ESLint (Antfu + UnoCSS presets), unbuild, UnoCSS.
- **Release/maintenance**: changelogithub, bumpp, taze, simple-git-hooks.

## Structure & Responsibilities

- **Root workspace**
  - `package.json`: root scripts for lint/typecheck/build/test across workspace.
  - `pnpm-workspace.yaml`: workspace package globs + shared version catalogs.
  - `uno.config.ts`: shared UnoCSS shortcuts/rules.
  - `vitest.config.ts`: root test discovery.
- **Docs app**
  - `docs/`: documentation site for all integrations and packages.
- **Packages**
  - `packages/integrations`: aggregate integration package.
  - `packages/vitepress-plugin-*`: VitePress plugin family.
  - `packages/markdown-it-*`: Markdown-It plugin family.
  - `packages/ui`, `packages/ui-asciinema`, `packages/ui-rive-canvas`: Vue UI components.
  - `packages/unconfig-vitepress`: VitePress config helpers.
- **Build/CI/Lint**
  - `.github/workflows`: CI pipelines.
  - `eslint.config.ts`: lint and formatting policy.

## Key Path Index (what lives where)

- `docs/`: VitePress docs app and demo content.
- `packages/integrations`: umbrella package for integrations.
- `packages/markdown-it-bi-directional-links`: bi-directional link plugin for Markdown-It.
- `packages/markdown-it-element-transform`: element transform plugin for Markdown-It.
- `packages/markdown-it-unlazy-img`: image optimization/lazy-loading helper plugin for Markdown-It.
- `packages/ui`: shared Vue UI components.
- `packages/ui-asciinema`: Asciinema Vue UI integration.
- `packages/ui-rive-canvas`: Rive Canvas Vue UI integration.
- `packages/unconfig-vitepress`: VitePress-oriented config utilities.
- `packages/vitepress-plugin-breadcrumbs`: breadcrumbs plugin.
- `packages/vitepress-plugin-enhanced-mark`: enhanced mark/highlight plugin.
- `packages/vitepress-plugin-enhanced-readabilities`: readability enhancement plugin.
- `packages/vitepress-plugin-git-changelog`: git changelog integration plugin.
- `packages/vitepress-plugin-graph-view`: graph view plugin.
- `packages/vitepress-plugin-highlight-targeted-heading`: targeted heading highlight plugin.
- `packages/vitepress-plugin-index`: page/content index plugin.
- `packages/vitepress-plugin-inline-link-preview`: inline link preview plugin.
- `packages/vitepress-plugin-meta`: meta extraction plugin.
- `packages/vitepress-plugin-og-image`: OG image generation plugin.
- `packages/vitepress-plugin-page-properties`: page properties plugin.
- `packages/vitepress-plugin-sidebar`: sidebar enhancement plugin.
- `packages/vitepress-plugin-thumbnail-hash`: thumbnail hash plugin.

## Commands (pnpm with filters)

> Use pnpm workspace filters to scope tasks. Replace the filter with a real workspace package name (e.g. `@nolebase/ui`, `@nolebase/vitepress-plugin-meta`, `@nolebase/markdown-it-unlazy-img`).

- **Install**
  - `pnpm install`
- **Typecheck**
  - Root: `pnpm typecheck`
  - Target package: `pnpm -F <package.json name> typecheck`
  - Example: `pnpm -F @nolebase/ui typecheck`
- **Unit tests (Vitest)**
  - Root: `pnpm test:run`
  - Target file: `pnpm exec vitest run <path/to/test-file>`
  - Target package: `pnpm -F <package.json name> exec vitest run`
- **Lint**
  - `pnpm lint`
  - `pnpm lint:fix`
- **Build**
  - Root: `pnpm build`
  - Target package: `pnpm -F <package.json name> build`
  - Example: `pnpm -F @nolebase/vitepress-plugin-meta build`

## Development Practices

- Favor clear module boundaries; shared logic goes in `packages/`.
- Keep runtime entrypoints lean; move heavy logic into services/modules.
- Prefer functional patterns + DI (`injeca`) for testability.
- Use Valibot for schema validation; keep schemas close to their consumers.
- Use Eventa (`@moeru/eventa`) for structured IPC/RPC contracts where needed.
- Use `errorMessageFrom(error)` from `@moeru/std` to extract error messages instead of manual patterns like `error instanceof Error ? error.message : String(error)`. Pair with `?? 'fallback'` when a default is needed.
- Do not add backward-compatibility guards. If extended support is required, write refactor docs and spin up another Codex or Claude Code instance via shell command to complete the implementation with clear instructions and the expected post-refactor shape.
- If the refactor scope is small, do a progressive refactor step by step.
- When modifying code, always check for opportunities to do small, minimal progressive refactors alongside the change.

## Styling & Components


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nolebase/integrations](https://github.com/nolebase/integrations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
