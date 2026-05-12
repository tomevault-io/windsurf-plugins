---
trigger: always_on
description: Code-first, single-schema CMS built inside Astro 6. Runtime, admin UI, routes, and middleware all live inside `src/cms/`. No external package boundary — own your code.
---

# Kide CMS

Code-first, single-schema CMS built inside Astro 6. Runtime, admin UI, routes, and middleware all live inside `src/cms/`. No external package boundary — own your code.

## Repo Structure

```
src/
  cms/
    cms.config.ts         # CMS configuration (collections, admin, locales)
    collections/          # Collection definitions
    adapters/             # db, email, storage adapters
    internals/            # Thin runner scripts + integration.ts + virtual.d.ts (not user-edited)
    migrations/           # Drizzle migrations
    .generated/           # Auto-generated schema/types/validators/api (do not edit)
    core/                 # CMS runtime library (define, api, auth, schema, content, ...)
    admin/                # Admin UI (components, layouts, lib)
    routes/               # Admin pages + API routes (injected by integration)
    middleware/           # Auth middleware (injected by integration)
    client/               # preview.ts — browser-side live-preview client
  components/             # App-level components (incl. BlockRenderer)
  layouts/, pages/, styles/, env.d.ts
```

(The Starlight docs site lives in a separate `kide-cms-docs` repo, not inside this project.)

## Commands

```bash
pnpm dev              # start dev server (auto-generates schema + pushes DB)
pnpm build            # production build
pnpm preview          # preview production build
pnpm check            # astro check (types) + eslint (lint)
pnpm format           # prettier --write .
pnpm cms:generate     # regenerate src/cms/.generated/ from cms.config.ts
pnpm cms:seed         # seed database with demo content
pnpm cms:admin        # create an admin user from CLI
```

## Validation (IMPORTANT)

After code changes, ALWAYS run:

1. `pnpm check` — Fix all errors before considering the task done.
2. `pnpm format` — Must be the very last step.

## Key Files

| File                               | Purpose                                               |
| ---------------------------------- | ----------------------------------------------------- |
| `src/cms/cms.config.ts`            | Top-level CMS config — collections, admin, locales    |
| `src/cms/collections/`             | Collection schemas                                    |
| `src/cms/adapters/`                | Project-specific db / email / storage adapters        |
| `src/cms/internals/`               | Thin runner scripts (runtime wiring, generator, etc.) |
| `src/cms/internals/integration.ts` | Astro integration (route injection, virtual modules)  |
| `src/cms/core/`                    | CMS runtime library (edit to change behavior)         |
| `src/cms/admin/`                   | Admin UI components, layouts, styles                  |
| `src/cms/routes/`                  | Admin pages + API routes injected by integration      |
| `src/cms/middleware/`              | Auth middleware injected by integration               |
| `src/cms/client/`                  | Browser-side live-preview client                      |

## Rules

- Use **pnpm** for all package management and scripts (`pnpm install`, `pnpm add`, `pnpm exec`, etc.). Do not use npm or yarn.
- Never edit `src/cms/.generated/` files — they're overwritten on every generation.
- DB columns use `snake_case`, TS fields use `camelCase`. System fields prefixed with `_`.
- Rich text is JSON AST, never HTML or Markdown in storage.
- Admin styles use shadcn CSS variables. Public site uses plain Tailwind colors — no shared styles.
- `labelField` on collections controls display name (fallback: title → name → first text field).
- Always query content through the typed local API (`cms.posts.findOne()`, `cms.pages.find()`, etc.) — never bypass it with raw DB queries or untyped wrappers.
- Routes in `src/cms/routes/` import app-specific code via `virtual:kide/*` modules (resolved by the integration's Vite aliases). Userland (`src/pages/`, `src/layouts/`, `src/components/`) imports directly via `@/cms/*` — virtual is the one-way core → user contract, don't use it in userland.
- Use the `cn()` utility from `@/cms/admin/lib/utils` for conditional class names — never use template literal interpolation for className.
- Import the CMS library via the `@/cms/core` alias (tsconfig `@/*` → `./src/*`), not relative paths.
- If you start dev server, remember to stop it when you're done.

## Stack

Astro 6, React 19, Drizzle ORM (SQLite dev), Zod, Tiptap, shadcn/ui, Tailwind CSS v4, PBKDF2 auth, nanoid, Sharp (image optimization), pnpm, Node >=22.12.0

## Field Types

All fields share base options: `label`, `description`, `required`, `defaultValue`, `indexed`, `unique`, `translatable`, `condition`, `admin`, `access`.

| Field      | Type-specific options                                                           |
| ---------- | ------------------------------------------------------------------------------- |
| `text`     | `maxLength?: number`                                                            |
| `slug`     | `from?: string` — field to auto-generate slug from                              |
| `email`    | _(base only)_                                                                   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mhernesniemi/kide-cms](https://github.com/mhernesniemi/kide-cms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
