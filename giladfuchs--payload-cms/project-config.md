---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Scope

- Next.js 16 (App Router) application backed by Payload CMS 3, using Postgres.
- Treat `other/` as out of scope: do not inspect, edit, format, lint, test, or reference files from that directory unless the user explicitly asks for it.
- Never read or expose values from `.env`; use `.env.example` to understand configuration.

## Development Commands

- Requires Node.js 24 and pnpm 10+.
- `pnpm dev` — start Next.js on port 3355.
- `pnpm build` — production build.
- `pnpm typecheck` — `tsc --noEmit`.
- `pnpm lint` / `pnpm lint:fix` — ESLint across the repo; for focused work, prefer `pnpm exec eslint <files>` scoped to changed files.
- `pnpm format` — Prettier across the repo; prefer `pnpm exec prettier --write <files>` scoped to changed files rather than running it repo-wide.
- `pnpm check` — typecheck + lint:fix + format, all repo-wide; mutates files, avoid unless the user asks for a full sweep.
- `pnpm generate:types` — regenerate `src/lib/core/types/payload-types.ts` (root `payload-types.ts` only re-exports it) after changing any Payload collection/global schema.
- `pnpm generate:importmap` — regenerate `src/app/(payload)/admin/importMap.js` after changing admin-facing custom components.
- `pnpm generate:all` — runs both of the above.
- `pnpm reset` and `pnpm seed` are **destructive** database commands (`payload migrate:fresh`, then `seed` also loads demo data via `seed/run.ts`). Only run when explicitly requested.
- There is no automated test suite; validate manually (dev server / admin UI) when practical.

## Architecture

### Data access layer (DAL) — swappable fetch strategy

`src/lib/core/dal/index.ts` exports a single `DAL` object that all data-fetching code depends on (via the `DalStatic` interface in `src/lib/core/types/types.ts`), never a concrete adapter directly. Two implementations exist and are swapped by editing which import is active in `index.ts`:

- `queries.ts` (default/active) — uses the local Payload SDK directly; Payload and Next.js share one runtime (monolith mode).
- `api.ts` — fetches through Payload's REST endpoints (`/api/...`) via `fetch()`, for headless/decoupled deployments. In this mode, `revalidateTag()` must NOT be called from inside Payload collection hooks — a separate Next.js revalidation endpoint must be triggered after mutations instead.
- `base-dal.ts` — shared `BaseDal` class both adapters extend, holding adapter-agnostic resolution logic for the Archive block (`populateBy: "collection"` vs `"selection"`) so the "which docs does this archive show" logic isn't duplicated. `resolvePageLayout` is typed with a `this: <required methods>` parameter, so it runs against whichever subclass (`Queries`/`Api`) calls it.

When adding data-fetching logic, extend the `DalStatic` interface and implement it in both adapters, or the inactive adapter silently drifts out of sync.

### Content model & caching

- Collections/globals live in `src/lib/collections/` (`Pages`, `Blog`, `BlogComments`, `Media`, `SeoMedia`, `GalleryMedia`, `Users`, `SiteSettings`) and are registered in `src/payload.config.ts` / re-exported from `src/lib/collections/index.ts`.
- `Media`, `SeoMedia`, and `GalleryMedia` are separate upload collections with different intended uses and generated image sizes (see comments in each file): `Media` is generic hero/content-block images with no size variants (editors crop manually); `SeoMedia` always generates `og` (1200x630) and `card` (480x320) variants for the SEO tab's meta image; `GalleryMedia` generates one small variant for the Gallery block. Use the matching upload relationship rather than treating them as interchangeable.
- `src/lib/collections/hooks.ts` implements tag-based revalidation: `makeRevalidateHooks(collection)` builds `afterChange`/`afterDelete` hooks that call `revalidateTag` on `${collection}-${slug}` (old and new slug on rename) plus the sitemap tag. Redirects have their own `revalidateRedirect`/`revalidateDeleteRedirects` keyed by the `from` field. Any schema or mutation change must preserve this tagging convention or cached pages will go stale.
- `mediaTransformUploadHook` (also in `hooks.ts`) intercepts image uploads, converts non-webp images to webp via `sharp`, and rewrites filename/size/dimensions on the upload record before it's persisted — applies to both local and cloud storage upload paths.
- Media storage is provider-swappable via `STORAGE_PROVIDER` (`vercel` | `s3` | unset = local disk), read through `src/lib/core/config.ts`. S3-compatible config supports AWS S3, Cloudflare R2, MinIO, etc.

### Blocks (page-builder pattern)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giladfuchs/payload-cms](https://github.com/giladfuchs/payload-cms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
