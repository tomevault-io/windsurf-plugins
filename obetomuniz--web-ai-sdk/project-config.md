---
trigger: always_on
description: This file is the single source of truth for **any AI agent** (Cursor, Claude Code, Copilot, Devin, etc.) working on this repository. Read it once at the start of a session, then keep it open as a reference.
---

# AGENTS.md

This file is the single source of truth for **any AI agent** (Cursor, Claude Code, Copilot, Devin, etc.) working on this repository. Read it once at the start of a session, then keep it open as a reference.

For human-facing context see the root [`README.md`](./README.md) and [`CONTRIBUTING.md`](./CONTRIBUTING.md), plus one README per package under `packages/<name>/README.md`.

---

## 1. What this is

**`web-ai-sdk`** is a small monorepo of building blocks for the Web's Built-in AI APIs. Each package is framework-agnostic by default and ships a React subpath adapter; future Vue / Svelte adapters slot in the same way.

| Package                 | Wraps                                                  | Status     |
| ----------------------- | ------------------------------------------------------ | ---------- |
| `@web-ai-sdk/webmcp`    | `navigator.modelContext` (W3C WebMCP)                  | Ported     |
| `@web-ai-sdk/translator`| `Translator` (Web Built-in AI Translator API)       | Ported     |
| `@web-ai-sdk/summarizer`| `Summarizer` (Web Built-in AI Summarizer API)       | Ported     |
| `@web-ai-sdk/prompt`    | `LanguageModel` (Web Built-in AI Prompt API)        | New        |
| `@web-ai-sdk/detector`  | `LanguageDetector` (Web Built-in AI Language Detection API) | New |
| `@web-ai-sdk/all`       | meta-package; re-exports the five above under one install | New     |

Two private workspace apps live under `apps/`: `apps/docs` (Astro Starlight docs site with live React demos) and `apps/landing` (the marketing/demo site). Run `pnpm docs` or `pnpm landing` after `pnpm build`.

Each package exposes:

- **Vanilla** (`@web-ai-sdk/<pkg>`); TS/DOM only, zero framework deps.
- **React** (`@web-ai-sdk/<pkg>/react`); small hook adapter that wraps the vanilla core. `react` is an optional peer dep.

Lifecycle layer on purpose. Each package only manages session lifetime, cleanup, feature detection, and the gnarly bits (block serialization in translator, skeleton extraction + caching in summarizer, safe register/unregister in webmcp). Framework adapters, polyfills, and UI primitives are opt-in subpaths, not bundled into the core packages.

---

## 2. Stack & runtime

- **TypeScript** strict, ESM (`"type": "module"`), `verbatimModuleSyntax`.
- **pnpm 9** workspaces (declared in `pnpm-workspace.yaml`).
- **tsup** for builds; emits ESM + CJS + `.d.ts` from `src/index.ts` and `src/react/index.ts`.
- **Vitest + happy-dom** for tests.
- **Biome** for lint + format (one tool, scoped to `packages/**` and `apps/**` `.ts`/`.tsx`). No ESLint, no Prettier.
- **Changesets** for versioning + publishing.
- **Node** `>= 20.19.1` (declared in `engines`; `.nvmrc` pins Node 24 for local dev).
- **Corepack** provisions the right pnpm version automatically from `package.json#packageManager`. No extra tooling required.

---

## 3. Folder map

```
.
├── packages/
│   ├── webmcp/
│   │   ├── src/
│   │   │   ├── index.ts          # vanilla core
│   │   │   ├── index.test.ts
│   │   │   └── react/
│   │   │       ├── index.ts      # React hook
│   │   │       └── index.test.tsx
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── tsup.config.ts
│   │   ├── vitest.config.ts
│   │   └── README.md
│   ├── translator/               # same shape; adds serialize.ts for DOM block walking
│   ├── summarizer/               # same shape; adds skeleton.ts + cache.ts
│   ├── prompt/                   # same shape; adds api.ts + cache.ts
│   ├── detector/                 # same shape; adds api.ts + cache.ts
│   └── sdk/                      # @web-ai-sdk/all meta-package; re-exports the five above
├── apps/
│   ├── docs/                     # Astro Starlight docs site
│   └── landing/                  # Vite + React marketing landing
├── .changeset/
├── .github/workflows/            # CI gate + release + Pages deploy
├── .nvmrc                        # pins Node 24 for local dev
├── biome.json                    # lint + format
├── package.json                  # workspace root + every pnpm script (incl. "packageManager" → pnpm 9.15.0)
├── pnpm-workspace.yaml
├── tsconfig.base.json            # shared compiler options
├── README.md                     # human onboarding entry point
├── CONTRIBUTING.md               # contribution flow
└── AGENTS.md                     # ← you are here
```

Package layout is intentionally uniform; open one package, you've seen them all.

---

## 4. Commands

All workflow commands are pnpm scripts in `package.json`.

| Task                              | Command                    |
| --------------------------------- | -------------------------- |
| Watch + rebuild every package     | `pnpm dev`                 |
| Boot the Starlight docs (`:6006`) | `pnpm docs`                |
| Boot the landing site (`:5173`)   | `pnpm landing`             |
| Build every package               | `pnpm build`               |
| Build the docs + landing apps     | `pnpm build:apps`          |
| Lint + format audit               | `pnpm lint`                |
| Auto-fix lint + format            | `pnpm lint:fix`            |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [obetomuniz/web-ai-sdk](https://github.com/obetomuniz/web-ai-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
