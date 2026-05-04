---
trigger: always_on
description: This file is read automatically by AI coding agents. It defines the project stack, directory conventions, tiered context protocol, and the feature-scoping checklist you must follow before building any feature.
---

# PopPopLM Agent Context Guide

This file is read automatically by AI coding agents. It defines the project stack, directory conventions, tiered context protocol, and the feature-scoping checklist you must follow before building any feature.

---

## Stack — Pinned Versions

| Layer            | Package               | Version                                      |
| ---------------- | --------------------- | -------------------------------------------- |
| Runtime          | Bun                   | latest                                       |
| Web Framework    | Hono                  | 4.12.7                                       |
| Validation       | Zod                   | **4.3.6** (v4 — breaking changes from v3)    |
| Authentication   | @hono/auth-js         | 1.1.1                                        |
| Auth Core        | @auth/core            | 0.34.3                                       |
| AI Framework     | @mastra/core          | **1.13.2** (v1.x — pre-1.0 APIs are invalid) |
| AI CLI           | mastra                | 1.3.12                                       |
| AI Hono Adapter  | @mastra/hono          | 1.2.4                                        |
| AI Memory        | @mastra/memory        | 1.8.2                                        |
| AI Storage       | @mastra/libsql        | 1.7.0                                        |
| AI Logging       | @mastra/loggers       | 1.0.2                                        |
| AI Observability | @mastra/observability | 1.5.0                                        |
| ORM              | drizzle-orm           | 0.45.1                                       |
| ORM Migrations   | drizzle-kit           | 0.31.9                                       |
| ORM+Zod Bridge   | drizzle-zod           | 0.8.3                                        |
| Database Client  | @libsql/client        | 0.17.0                                       |
| AI Provider      | @ai-sdk/openai        | 3.0.47                                       |
| AI Embeddings    | @xenova/transformers  | 2.17.2                                       |
| Ingestion HTML   | cheerio               | 1.2.0                                        |
| Ingestion PDF    | pdf-parse             | 2.4.5                                        |
| Ingestion Media  | youtube-transcript    | 1.3.0                                        |
| CSS Framework    | tailwindcss           | **4.2.1** (v4 — CSS-first, no JS config)     |
| UI Components    | daisyui               | **5.5.19** (v5 — @plugin in CSS)             |
| Client Behavior  | alpinejs              | **3.15.8** (directive syntax is strict)       |
| Alpine Types     | @types/alpinejs       | **3.13.11** (TypeScript typings only)         |
| Icons            | @iconify/tailwind4    | **1.2.3** (CSS-first plugin)                 |
| Icon Set         | @iconify-json/lucide  | 1.2.97                                       |
| Hypermedia       | htmx.org              | **2.x** (v2 — event syntax changed)          |
| Route Validation | @hono/zod-validator   | 0.7.6                                        |
| Testing          | Bun test + happy-dom  | built-in / 20.8.4                            |


`package.json` is the absolute source of truth for versions.

---

## Directory Conventions

```
PopPopLM/
  src/
    index.tsx           ← Hono app entry point
    styles.css          ← Tailwind + DaisyUI + Iconify config (source)
    db/
      index.ts          ← drizzle client singleton
      schema.ts         ← all table definitions
      queries/          ← typed query helpers per domain
    lib/                ← shared helpers and pure logic
    types/              ← shared TS types
    routes/             ← Hono route groups (one file per domain or `<name>/index.tsx`)
    mastra/
      index.ts          ← Mastra instance
      agents/           ← one file per agent
      tools/            ← one file per tool
      workflows/        ← one file per workflow
    views/              ← full-page Hono JSX views
    components/         ← reusable HTMX partial components
  public/
    styles.css          ← compiled CSS output (do not edit)
  drizzle/              ← generated migration files
```

---

## Tiered Context Protocol

Apply context in tiers. Never attach all docs simultaneously — it degrades output quality.

### Tier 1 — Always Active (automatic via skills)

These rules are auto-applied by Antigravity via the skills in `.agents/skills/`:

| Skill               | Scope                                                     |
| ------------------- | --------------------------------------------------------- |
| `project (Standards)` | Every file — version pins, TypeScript, Check-Before-Write |
| `mastra`            | `src/mastra/**` — agent/tool/workflow patterns            |
| `database`          | `src/db/**` — schema, queries, migrations                 |
| `ui`                | `src/views/**`, `src/components/**`, `*.css` — UI layer   |
| `ingestion`         | `src/lib/ingest-source-content.ts`, `src/routes/sources.tsx`, `src/db/queries/sources.ts` — Source parsing |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PopPopRetired/PopPopLM](https://github.com/PopPopRetired/PopPopLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
