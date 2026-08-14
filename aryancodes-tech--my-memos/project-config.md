---
trigger: always_on
description: > **Audience:** Cursor agents, Claude Code, Copilot Workspace, and human reviewers evaluating AI-assisted engineering depth (e.g. Forward Deployed Engineer workflows).
---

# AGENTS.md - MyMemos AI Operating Manual

> **Audience:** Cursor agents, Claude Code, Copilot Workspace, and human reviewers evaluating AI-assisted engineering depth (e.g. Forward Deployed Engineer workflows).
>
> **Product:** **MyMemos** - local-first personal knowledge OS that replaces your browser's New Tab. Treat **MyMemos** / `mymemos` as canonical.

This document is the **source of truth** for how an AI agent should reason about, modify, and verify this codebase. It complements human docs (`README.md`, `CONTRIBUTING.md`) with machine-oriented invariants, decision trees, and verification contracts.

---

## 1. Repository topology (three surfaces, one product)

```
┌──────────────────────────────────────────────────────────────────────────┐
│ SURFACE A - Landing site (`src/`, TanStack Start, React 19, Tailwind 4) │
│   Routes: `/` (marketing + download), SSR via Nitro                      │
│   Constants: `@/lib/constants` → `shared/constants.ts`                 │
│   Do NOT import extension code directly                                │
└───────────────────────────────┬──────────────────────────────────────────┘
                                │ `/demo/` embeds built SPA
┌───────────────────────────────▼──────────────────────────────────────────┐
│ SURFACE B - Web demo (`extension/` → `public/demo/`, React 19)           │
│   Entry: `extension/index.html`, build: `vite.web.config.ts`             │
│   Settings: `localStorage` · Pages: IndexedDB · Attachments: OPFS        │
│   (separate origin from extension - data does not sync)                │
└───────────────────────────────┬──────────────────────────────────────────┘
                                │ same `extension/src/` source
┌───────────────────────────────▼──────────────────────────────────────────┐
│ SURFACE C - Browser extension (`extension/` → `dist/`, MV3 + CRXJS)     │
│   Entry: `extension/newtab.html`, overrides New Tab                    │
│   Settings: `chrome.storage.local` · Pages: IndexedDB · Attachments: OPFS│
└──────────────────────────────────────────────────────────────────────────┘
```

### Path alias warning

Aliases resolve differently depending on which `tsconfig` is active:

| Package        | Alias `@/` →      | Alias `@shared/` → |
| -------------- | ----------------- | ------------------ |
| Root / landing | `src/*`           | `shared/*`         |
| Extension      | `extension/src/*` | `../shared/*`      |

Before adding imports, confirm which package you are editing. Product constants: edit `shared/constants.ts` only.

### Generated artifacts - never hand-edit

| Path                                                         | Produced by                                                 |
| ------------------------------------------------------------ | ----------------------------------------------------------- |
| `public/demo/**`                                             | `npm run build:app` (extension web build)                   |
| `public/mymemos-extension.zip`                               | `npm run package:extension`                                 |
| `public/robots.txt`, `public/sitemap.xml`, `public/llms.txt` | `npm run generate:seo` (also `predev:web` / `prebuild:web`) |
| `src/routeTree.gen.ts`                                       | TanStack Router codegen                                     |
| `extension/dist/**`                                          | `npm run dev` or `npm run build:extension`                  |
| `.output/**`, `dist/**` (root)                               | `npm run build:web`                                         |

---

## 2. Mandatory pre-change protocol

Every non-trivial agent session **must** follow this sequence before writing code:

### Phase A - Classify the task

| Task class               | Examples                                   | Primary docs                                                      |
| ------------------------ | ------------------------------------------ | ----------------------------------------------------------------- |
| **Extension product**    | Sidebar, editor, storage, themes, search   | `extension/README.md`, `.cursor/rules/extension-architecture.mdc` |
| **Landing / marketing**  | Hero, scroll video, download flow          | `src/routes/README.md`, `.cursor/rules/landing-site.mdc`          |
| **Landing SEO / GEO**    | Meta tags, FAQ schema, `llms.txt`, sitemap | `src/lib/seo.ts`, `.cursor/SKILLS.md` → `landing-seo`             |
| **Build / CI / tooling** | Vite, workflows, scripts                   | `package.json`, `.cursor/rules/testing-ci.mdc`                    |
| **Cross-cutting**        | Constants, naming, security                | This file §3–§5                                                   |

### Phase B - Trace data flow

Ask explicitly:

1. **Where does state live?** (Zustand vs IndexedDB vs chrome.storage/localStorage)
2. **What gets persisted?** (Block JSON only - see §3.1)
3. **Which surface(s) are affected?** (extension-only, web demo, landing, or build pipeline)
4. **Is there an existing pattern?** (grep siblings before inventing abstractions)

### Phase C - Scope the diff


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aryancodes-tech/my-memos](https://github.com/aryancodes-tech/my-memos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
