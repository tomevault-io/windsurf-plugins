---
trigger: always_on
description: Nuxt 4 app for building, previewing, and exporting [Nuxt UI v4](https://ui.nuxt.com) themes: semantic palettes, per-role shades, neutral scale, radius, font, and independent light/dark token overrides, previewed live across real component, block, and template pages. Also ships AI theme generation (BYOK), design tools, and a Nuxt Content learning hub. Live at https://nuxt-ui-themes.com.
---

# Nuxt UI Theme Builder

Nuxt 4 app for building, previewing, and exporting [Nuxt UI v4](https://ui.nuxt.com) themes: semantic palettes, per-role shades, neutral scale, radius, font, and independent light/dark token overrides, previewed live across real component, block, and template pages. Also ships AI theme generation (BYOK), design tools, and a Nuxt Content learning hub. Live at https://nuxt-ui-themes.com.

Stack: Nuxt 4 · Vue 3 `<script setup lang="ts">` · Nuxt UI v4 · Tailwind CSS v4 · Pinia (+ persistedstate) · VueUse · Nuxt Content v3 · zod · AI SDK · nuxt-security · Vitest · Playwright. Node 22.22+, pnpm 10 (enforced), TypeScript 6 (typescript-eslint doesn't support 7 yet), ESLint 10.

## Commands

```bash
pnpm install          # postinstall runs `nuxt prepare`; .nuxt/ is required by eslint.config.mjs and typecheck
pnpm dev              # http://localhost:3000
pnpm lint             # ESLint (TS/Vue/JS)      · pnpm lint:fix
pnpm format:check     # Prettier (JSON/MD/YAML/CSS only) · pnpm format
pnpm typecheck        # nuxt typecheck (vue-tsc), ~1 min
pnpm test             # Vitest unit suite, ~70 s · pnpm vitest run <path> for one file
pnpm test:e2e:smoke   # Playwright smoke suite (what CI runs)
pnpm workflow:full    # lint:fix → format → typecheck → test
pnpm knip --include-entry-exports   # unused exports (plain `pnpm knip` treats every file as an entry)
```

Font-provider `403`/"Could not fetch" warnings during build/test come from sandboxed network access and are harmless.

## Architecture

```
app/app.vue              Picks the layout: coming-soon | preview (?preview query or /preview shell) | default
app/layouts/default.vue  Editor shell: sidebar editor + page area (iframe preview for demos) + source view + command palette
app/layouts/preview.vue  What renders inside the iframe; syncs with the parent via postMessage
app/stores/theme.ts      Single source of truth: ThemeConfig, undo/redo history, saved presets
app/types/theme.ts       ThemeConfig types + ThemeConfigSchema (zod); re-exports shared constants
shared/constants/        Cross-layer constants (palettes, fonts, shades, routes, AI fallbacks)
app/composables/         useThemeApply, useThemeExport, usePreviewIframe, useSourceCode, useAi*, …
app/utils/navigation/    Sidebar / command-palette registry for every preview route
app/utils/cssGenerator.ts, defaults.ts, presets.ts, paletteGenerator.ts, customPalettes.ts, iframeProtocol.ts, seoDescriptions.ts
app/pages/               components/[slug], blocks/[slug] (data-driven), templates/*.vue, tools/, learn/
server/api/              ai/generate.post.ts, auth/launch.post.ts, highlight.post.ts, source/[...path].get.ts
modules/source-code-embed.ts  Embeds app/pages/**/*.vue as a Nitro virtual module for /api/source
content/learn/           Nuxt Content articles (schema in content.config.ts)
```

Only demo routes use the preview iframe (`isFramedRoute` in `shared/constants/routes.ts`: component, block, and template pages, plus `/ai`). Every other page renders directly in the editor so search engines see its content; the iframe document is `noindex`. Preview iframes start on the prerendered `/preview` shell and are moved to the current route with `NAVIGATE` after the ready handshake, so the theme is applied before the demo page renders.

Data flow: editor controls call store setters → `useThemeApply()` writes semantic palette names to `useAppConfig().ui.colors` **and** injects CSS variables (radius, font, shade shifts, token overrides, dark-mode deltas) via `useHead` → the default layout posts `THEME_SYNC` / `COLORMODE_SYNC` to the iframe → `preview.vue` applies them with `store._syncConfig()` (no history) → export composables serialize the config without mutating runtime state.

## Guardrails

- **Layout selection lives in `app.vue`.** `<NuxtLayout :name>` overrides page meta, so `definePageMeta({ layout })` has no effect. Demo pages render inside the iframe with the `preview` layout; everything else renders directly in `default.vue`.
- **Theme store**: every user-facing mutation pushes history; iframe sync uses `_syncConfig` (no history). Anything loaded from outside (persisted state, imports, presets, AI output, postMessage) goes through `ThemeConfigSchema`.
- **Persistence is split on purpose.** `config` + `activePresetName` go in the `theme` cookie (readable during SSR, ~2 KB). `savedPresets` go in localStorage under `theme-presets`. Browsers silently drop cookies over 4096 bytes, and one saved preset is enough to push the whole store past that. After hydration the store calls `_resetHistory()` so undo can't jump back to `DEFAULT_THEME`.
- **Client-only state must not change SSR output.** Gate anything rendered from localStorage behind `useMounted()` or `<ClientOnly>` (see `EditorSavedThemes`, `EditorToolbar`), otherwise hydration mismatches.
- **Keep the two-strategy theme apply** (app config colors + injected CSS variables). Don't collapse them.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mattycraig/nuxt-theme-builder](https://github.com/mattycraig/nuxt-theme-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
