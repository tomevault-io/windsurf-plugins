---
trigger: always_on
description: **Rasengan.js** is a modern React meta-framework built on top of **Vite** and **react-router**. It provides SSR, SSG, SPA modes, file-system routing, config-based routing, MDX support, and an adapter system for deployment (Vercel, Node).
---

# Rasengan.js — AI Agent Codebase Guide

## 1. Project Overview

**Rasengan.js** is a modern React meta-framework built on top of **Vite** and **react-router**. It provides SSR, SSG, SPA modes, file-system routing, config-based routing, MDX support, and an adapter system for deployment (Vercel, Node).

- **Author:** Dilane Kombou (dilane3)
- **License:** MIT
- **Homepage:** https://rasengan.dev
- **Repo:** https://github.com/rasengan-dev/rasenganjs.git
- **Node requirement:** >=22.12.0
- **Package manager:** pnpm (v10.9.0)

---

## 2. Monorepo Structure (pnpm workspace)

```
rasenganjs/
  packages/
    rasengan/              # Core framework (routing, SSR, SSG, CLI, Vite plugin, server)
    create-rasengan/       # CLI scaffolding tool (`npx create-rasengan`)
    rasengan-shuriken/     # Template cloning CLI (`npx shuriken`)
    rasengan-kurama/       # Lightweight Zustand-like state management
    rasengan-mdx/          # MDX plugin & components for Rasengan
    rasengan-serve/        # Production Express server for built apps
    rasengan-vercel/       # Vercel deployment adapter
    rasengan-i18n/         # Internationalization library
    rasengan-image/        # Optimized Image component (lazy, blur/wave placeholder)
    rasengan-theme/        # Light/dark/system theme management
    rasengan-kage-demo/    # Interactive feature tour / onboarding library
  docs/                    # The rasengan.dev documentation site (itself a Rasengan app)
  playground/              # Example/test projects
  examples/                # shadcn example
  scripts/                 # Build, pack, release, CI scripts
```

---

## 3. Core Package: `rasengan` (v1.2.1)

### 3.1 Entry Points (exports)

| Path                    | Purpose                                                                                                            |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `rasengan`              | Main: routing utilities + `defineConfig`                                                                           |
| `rasengan/client`       | Client entry: `renderApp()`                                                                                        |
| `rasengan/server`       | Server entry: `createRequestHandler`, `resolveBuildOptions`, `detectRuntime`, re-exports `express` + `compression` |
| `rasengan/plugin`       | Vite plugin: `rasengan()`, `Adapters`, `plugins[]`                                                                 |
| `rasengan/dynamic`      | Code-splitting: `dynamicLoad()`, `lazyLoadPage()`                                                                  |
| `rasengan/types/client` | Ambient types for `.mdx`, `.md`, virtual modules                                                                   |

### 3.2 CLI (`rasengan` command, via `bin.js`)

Uses **Commander**. Two commands:

- **`dev`** — Starts Vite dev server with Express middleware (HMR, SSR, port 5320 by default)
- **`build`** — Runs Vite build (client + SSR + optional SSG), writes `config.json`

### 3.3 Vite Plugin System

The main plugin `rasengan()` (in `src/core/plugins/index.ts`) orchestrates the entire build lifecycle:

| Plugin                     | Virtual Module                | Purpose                                                                                        |
| -------------------------- | ----------------------------- | ---------------------------------------------------------------------------------------------- |
| `rasengan()`               | —                             | Main build plugin: loads config, writes `config.json`, handles prerender/SSG, triggers adapter |
| `loadRasenganGlobal()`     | —                             | Injects `Rasengan` global constant (version, ssr flag)                                         |
| `rasenganConfigPlugin()`   | `virtual:rasengan-config`     | Exposes partial config (server + redirects) at runtime                                         |
| `flatRoutesPlugin()`       | `virtual:rasengan/router`     | Generates Router from `import.meta.glob` on `_routes/`                                         |
| `buildOutputInformation()` | `virtual:rasengan/build-info` | Provides `resolveBuildOptions()` at build time                                                 |
| `fixCPathPlugin()`         | —                             | Fixes Windows C: drive path resolution                                                         |

The `plugins[]` array at the bottom of the file provides the default plugin set (excluding `rasenganConfigPlugin` and `buildOutputInformation` which are used in `vite.config.ts`).

### 3.4 Configuration (`rasengan.config.js`)

Users define their config using `defineConfig()` which returns an `AppConfig`:

```ts
interface AppConfig {
  ssr?: boolean; // default: true
  prerender?: boolean | { routes: string[] }; // SSG mode
  sageMode?: { reactCompiler?: boolean | { compilationMode: 'annotation' } };
  server?: { development?: { port?: number; open?: boolean } };

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rasengan-dev/rasenganjs](https://github.com/rasengan-dev/rasenganjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
