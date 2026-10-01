---
trigger: always_on
description: Astro Starlight development standards for aspire.dev documentation site
---


# aspire.dev Development Instructions

This is the aspire.dev documentation site, built with [Astro Starlight](https://starlight.astro.build/). Frontend source lives under `src/frontend/`.

Astro guidance is adapted from [Awesome GitHub Copilot's Astro development instructions](https://awesome-copilot.github.com/instruction/astro/) for this repository. Use `src/frontend/package.json`, `astro.config.mjs`, and `tsconfig.json` as the source of truth for installed versions and configuration; do not introduce optional upstream features without a task that requires them.

## Project Stack

- **Astro 7.x** with **Starlight** documentation theme and the **Content Layer API**
- **TypeScript** (strict mode, `astro/tsconfigs/strict`)
- **Static site generation** (SSG) with selective client-side interactivity
- **pnpm** as the package manager (`pnpm install`, `pnpm dev`)
- **15 locales** with Lunaria translation tracking

## Running Locally

```bash
cd src/frontend
pnpm install
pnpm dev        # starts dev server at http://localhost:4321
```

Search is disabled in dev mode. Running `pnpm dev` is sufficient to verify documentation rendering changes. Prefer CI for production builds and search validation; never run `pnpm build` locally without explicit user permission. Use `pnpm preview` when a production build is already available.

## Astro Architecture and Type Safety

- Render content at build time in `.astro` frontmatter by default. Keep the static-site architecture; Astro Actions, sessions, server islands (`server:defer`), and on-demand API routes require server runtime support and are not drop-in additions to this site.
- Prefer existing `.astro` components and browser scripts or Web Components for interactivity. Add a UI framework only when the task needs it; hydrate framework islands selectively with `client:load`, `client:idle`, or `client:visible`.
- Preserve `astro/tsconfigs/strict` and the generated `.astro/types.d.ts` include. Run `pnpm exec astro sync` from `src/frontend` after changing collections or Astro configuration; do not edit generated types.
- Define component props with a TypeScript `Props` interface, use explicit defaults where appropriate, and keep components focused and composable.
- Write fully closed HTML with valid nesting. Astro 7 rejects unclosed tags rather than repairing them.
- Account for Astro 7's default JSX-style whitespace handling (`compressHTML: 'jsx'`): use explicit `{' '}` between inline elements when a visible space is required.

## Client-Side Navigation

The existing `src/components/starlight/Head.astro` override renders `<ClientRouter />` from `astro:transitions`. Reuse it rather than adding another router.

- Initialize page-specific browser behavior on `astro:page-load` so it also works after client-side navigation.
- Make initialization idempotent and clean up listeners, observers, and timers when their page elements are replaced; avoid duplicate handlers after repeated navigation.
- Use `transition:persist` only for elements whose state should intentionally survive navigation.
- Keep content and links usable without JavaScript, and verify interactive changes on both an initial load and a client-side navigation.

## Two-slash TypeScript Examples

Use the `twoslash-validator` skill whenever you add or edit a `twoslash` TypeScript code fence, TypeScript AppHost sample, or generated TypeScript API data. The site must not ship rendered two-slash diagnostics; run `pnpm test:unit:twoslash-blocks` from `src/frontend` and fix every reported diagnostic instead of adding allowlists or suppressions.

## Project Structure

```
src/frontend/
├── astro.config.mjs          # Starlight config, plugins, integrations
├── ec.config.mjs              # Expressive Code config (themes, plugins)
├── tsconfig.json              # TypeScript config with path aliases
├── config/                    # Sidebar topics, locales, redirects, cookies, SEO head
│   └── sidebar/               # Sidebar topic modules (7 files)
├── src/
│   ├── content.config.ts      # Content collections (docs, i18n, packages)
│   ├── route-data-middleware.ts
│   ├── assets/                # Images, icons, logos
│   ├── components/            # Custom Astro components
│   │   └── starlight/         # Starlight component overrides
│   ├── content/docs/          # All documentation pages (MDX/MD)
│   ├── data/                  # JSON data files + pkgs/ API reference
│   ├── expressive-code-plugins/  # Custom EC plugins (disable-copy)
│   ├── pages/                 # Astro page routes
│   ├── styles/                # Global CSS (site.css)
│   └── utils/                 # Helpers, package utils, sample tags
```

## Import Aliases

Always use these path aliases (defined in `tsconfig.json`) instead of relative paths:

| Alias | Resolves to |
|---|---|
| `@assets/*` | `./src/assets/*` |
| `@components/*` | `./src/components/*` |
| `@data/*` | `./src/data/*` |
| `@scripts/*` | `./src/scripts/*` |
| `@utils/*` | `./src/utils/*` |

Example usage in MDX frontmatter imports:

```mdx
import LearnMore from '@components/LearnMore.astro';
import ThemeImage from '@components/ThemeImage.astro';
import { Aside, Code, Steps, LinkButton, Tabs, TabItem } from '@astrojs/starlight/components';
```

## Starlight Plugins


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/aspire.dev](https://github.com/microsoft/aspire.dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
