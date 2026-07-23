---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Keycloak login theme built with React 19, TypeScript, Tailwind CSS v4, shadcn/ui, and Keycloakify v11. It produces a `.jar` file deployed to Keycloak's `providers/` directory. There are no automated tests — Storybook is the primary development and visual testing environment.

## Commands

```bash
pnpm dev                   # Vite dev server for UI development
pnpm storybook             # Storybook on port 6006 (primary dev workflow)
pnpm build                 # TypeScript check + Vite build
pnpm build-keycloak-theme  # Full build → outputs .jar in dist_keycloak/
pnpm build-storybook       # Build static Storybook to storybook-static/
pnpm format                # Prettier (write)
pnpm emails:preview        # Preview email templates locally
pnpm emails:check          # Validate email templates
```

After adding new Keycloak env vars or pages, regenerate the auto-generated file:
```bash
pnpm exec keycloakify update-kc-gen
```

`postinstall` automatically runs `keycloakify sync-extensions` — no manual step needed after `pnpm install`.

## Architecture

### Entry Points

- `src/main.tsx` — browser dev entry (uses mock `kcContext`)
- `src/main-kc.tsx` — production Keycloak entry (reads `window.kcContext`)
- `src/main-kc.dev.tsx` — Keycloak dev entry with HMR support
- `src/kc.gen.tsx` — **auto-generated** by `keycloakify update-kc-gen`; exports `KcEnvName`, `ThemeName`, `KcPage`, `kcEnvDefaults`. Do not edit manually.

### Login Theme (`src/login/`)

**Provider tree** (`KcPage.tsx`):
```
KcContextProvider → I18nProvider → KcClsxProvider (doUseDefaultCss: false) → ThemeProvider → PageIndex
```

**Page routing** (`pages/PageIndex.tsx`): Switch on `kcContext.pageId` (e.g. `"login.ftl"`) to lazy-loaded page components. Each page lives in `pages/{page-name}/` with an `index.ts` barrel, `Page.tsx`, and optional `Page.stories.tsx`.

**Page pattern**: Each `Page.tsx` calls `assert(kcContext.pageId === "...")` then renders `<Template>` with children.

**Template system** (`components/Template/`):
- `Template.tsx` reads `kcContext.properties.SHADCN_THEME_LAYOUT` and renders one of three layout components from `layouts/`: `TwoColumnLayout`, `CenteredCardLayout`, or `ImageAsideLayout`.
- `useApplyThemePreset.ts` writes CSS custom properties to `:root` at runtime based on `SHADCN_THEME_PRESET`, `SHADCN_THEME_BASE`, `SHADCN_THEME_RADIUS`, and `SHADCN_THEME_FONT` from `kcContext.properties`.
- Theme tokens are defined in `theme/Themes.ts` as OKLCH values split into `basePalettes` (neutral surface system) and `themePresets` (primary accent color). These two are layered: base handles backgrounds/borders/muted, preset handles primary/button colors.
- Default values for all env vars live in `components/Template/Defaults.ts`.

**Context extension** (`KcContext.ts`): Extends the base Keycloakify context to add `properties: Record<KcEnvName, string>` (typed env vars), `darkMode?: boolean`, and `client.baseUrl`.

**i18n** (`i18n.ts`): Uses `i18nBuilder` from `@keycloakify/login-ui/i18n` with custom translation keys across 30 locales. Add new translation keys here using `.withCustomTranslations({...})`.

**Asset URLs** (`shared/resolveAssetUrl.ts`): Handles the `%BASE_URL%/filename` pattern for self-hosted assets placed in `public/`. Use this helper when rendering logo/image URLs from `kcContext.properties`.

**Style injection** (`styleLevelCustomization.tsx`): Sets `doUseDefaultCss: false` (disables Keycloakify's default styles) and wraps children in `ThemeProvider` for dark/light/system mode. Imports `./index.css` which is the main Tailwind entry point.

**Password confirmation toggle** (`components/UserProfileFormFields/DO_MAKE_USER_CONFIRM_PASSWORD.ts`): A single boolean export that controls whether the registration form requires users to re-enter their password. Change this file to enable/disable the confirmation field.

### Email Templates (`src/email/`)

Built with `jsx-email` and compiled during `build-keycloak-theme` via the `keycloakify-emails` Vite plugin's `postBuild` hook in `vite.config.ts`. Templates are in `templates/`. Email i18n uses a **separate system** from login i18n: `src/email/i18n.ts` uses `i18next`/`react-i18next` with JSON translation files in `src/email/locales/{locale}/translation.json` (not the keycloakify i18nBuilder).

### Shared Components (`src/components/`)

shadcn/ui components and `ThemeProvider`. Uses `@/` path alias for `src/`.

### Storybook (`.storybook/`)

`preview.ts` injects a global decorator that mirrors all Keycloak env vars as toolbar controls (locale, layout, theme preset, base palette, radius, font, placeholder). Stories accept `kcContext` as a prop and are mocked via `src/login/mocks/getKcContextMock.ts`.

## Environment Variables (SHADCN_THEME_*)

Valid values for each Keycloak env var (configured in `vite.config.ts` → `environmentVariables`):

| Var | Valid values |
|-----|-------------|
| `SHADCN_THEME_LAYOUT` | `two-column` (default) · `centered-card` · `image-aside` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Oussemasahbeni/keycloakify-shadcn-starter](https://github.com/Oussemasahbeni/keycloakify-shadcn-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
