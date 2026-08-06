---
trigger: always_on
description: This document describes the **OWD client monorepo**: what it is, how it is organized, and how to extend it. Include `@AGENTS.md` in new sessions so assistants do not need the layout explained from scratch.
---

# Open Web Desktop (OWD) — Agent & contributor context

This document describes the **OWD client monorepo**: what it is, how it is organized, and how to extend it. Include `@AGENTS.md` in new sessions so assistants do not need the layout explained from scratch.

**Product positioning:** an open, modular **Nuxt module** for building **browser-based desktop experiences** (windows, shell, workspace). Themes shape the “OS” look and feel; apps register programs via `defineDesktopApp`. The repo is meant to be reusable, documented, and approachable for contributors and downstream teams.

**Module playground playbook** (Nuxt module authoring, `nuxt-module-build`, GitHub Pages): [`docs/agents/OWD_APP_MODULE_PLAYGROUND.md`](docs/agents/OWD_APP_MODULE_PLAYGROUND.md). In Cursor, see also `.cursor/rules/owd-app-module-playground.mdc` under `apps/**` and `themes/**`.

---

## One-sentence architecture

A single Nuxt app (`desktop/`) loads **`@owdproject/core`**, which reads **`desktop.config.ts`**, installs **theme → optional modules → apps** (all as Nuxt modules), and provides the shared runtime (Pinia, application/window management, core UI). Themes customize layout and chrome; apps register desktop programs through **`defineDesktopApp`**.

---

## Tech stack

| Layer | Choice |
|--------|--------|
| UI | **Vue 3**, **Nuxt 4** (monorepo desktop is typically SPA: `ssr: false`) |
| Monorepo | **pnpm** workspaces + **Nx** (`nx run desktop:…`) |
| State | **Pinia** |
| Styling | **Tailwind** via `@owdproject/kit-tailwind`; **PrimeVue** via `@owdproject/kit-primevue` (theme installs the kit for its UI stack — core 3.4+ does not) |
| Icons / fonts | **@nuxt/icon**, **@nuxt/fonts** |
| i18n | **@nuxtjs/i18n** |
| Extension packages | **@nuxt/module-builder** → published **`dist/module.mjs`** |

---

## Repository map (root = `client/`)

```
client/
├── desktop/                 # Monorepo Nuxt shell (primary dev entry)
├── packages/
│   ├── core/                # @owdproject/core — orchestration module + desktop CLI
│   ├── module-fs/           # Optional ZenFS virtual filesystem
│   ├── module-docs/         # Optional in-app documentation (Nuxt Content)
│   ├── module-persistence/  # Optional Pinia persistence (IndexedDB)
│   ├── kit-primevue/        # Optional locally — PrimeVue + tailwindcss-primeui (PV themes)
│   ├── kit-tailwind/        # Optional locally — Tailwind content registration (apps/themes)
│   ├── kit-nuxt-ui/         # Optional external — Nuxt UI stack (no PrimeVue)
│   └── nx/                  # Nx plugin helpers (submodule)
├── apps/                    # Desktop apps (@owdproject/app-*)
├── themes/                  # Desktop themes (@owdproject/theme-*)
├── plugins/                 # Optional workspace slot for future Nuxt plugins
├── template/                # Generated starter (npm create owd) — do not hand-edit
├── packages/core/template-blueprint/  # Maintainer source for template/
├── package.json
├── pnpm-workspace.yaml
└── nx.json
```

**Folder convention:** there is **no** top-level `modules/` directory. Shared packages live under **`packages/*`**. The **`modules`** field in `desktop.config.ts` is a **list of Nuxt package names** to load, not a filesystem path.

### `desktop/` — monorepo shell

- **`nuxt.config.ts`** — registers `@owdproject/core`, Nuxt options (host, i18n, `workspaceDir`, …).
- **`desktop.config.ts`** — declarative desktop config (`defineDesktopConfig`): `theme`, `modules`, `apps`.
- **`app/app.vue`** — minimal root; renders the active theme (e.g. `<Desktop />`).
- Window/app logic lives in **core + theme + loaded apps**, not in `desktop/` itself.

### `packages/core/` — `@owdproject/core`

Central Nuxt module and **`desktop`** CLI (`bin/desktop.js`; `owd` is a deprecated alias).

**Startup (logical order):**

1. Initialize `runtimeConfig.public.desktop`.
2. Dynamic import of `desktop.config.ts` from the Nuxt root (`rootDir + '/desktop.config.ts'`). Legacy `owd.config.ts` is still accepted with a console warning. Invalid or missing config **stops** setup.
3. Merge desktop config into `runtimeConfig.public.desktop` (includes **`coreVersion`** from core’s `package.json`).
4. **`installModule`** in order: **theme** → **`modules`** → **`apps`**.
5. Core installs Pinia, fonts, icons, VueUse, i18n, … **Tailwind and PrimeVue are not installed by core** (since 3.4). The active **theme** installs its UI kit (`kit-primevue` for PV themes, or `kit-nuxt-ui` for `@owdproject/theme-nuxt`).
6. Global core components, client plugins, auto-imports from `composables`, `stores`, `utils` (not `runtime/internal/` — kernel controllers are private).

**`runtime/internal/`** — private kernel (`ApplicationController`, `WindowController`). Not auto-imported; use `defineDesktopApp` and `useApplicationManager` instead.

**There is no `packages/core/playground/`.** Develop core through the monorepo **`desktop/`** app, or run `pnpm run dev:prepare` in `packages/core` to stub `dist/` only.

**Public API** (`packages/core/index.ts`, `types/`):

- **`defineDesktopConfig`** — used in `desktop.config.ts`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [owdproject/client](https://github.com/owdproject/client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
