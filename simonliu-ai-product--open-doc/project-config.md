---
trigger: always_on
description: You are working on the **open-doc framework** — the runtime, CLI, and tooling that ship to npm.
---

# open-doc — Framework Repo Guide

You are working on the **open-doc framework** — the runtime, CLI, and tooling that ship to npm.

(Document-authoring guidance lives in the `create-doc` / `doc-authoring` skills under `packages/core/skills/`. Those are for editing files inside `docs/`, not for editing the framework.)

## Layout

pnpm + Turbo monorepo.

| Path | Package | Role |
| --- | --- | --- |
| `packages/core` | `@open-document/core` | Runtime (document browser, page viewer, outline, themes, assets panel, design panel, PDF/HTML export), Vite plugins, dev API, `open-doc` dev/build CLI, canonical skills. |
| `packages/cli` | `@open-document/cli` | `npx @open-document/cli init` scaffolder + project template. |
| `packages/mcp` | `@open-document/mcp` | MCP server exposing the `ops` layer as tools over Streamable HTTP. Opt-in; mounted at `/mcp` by `open-doc dev --mcp`. |
| `apps/demo` | private | Local consumer of `@open-document/core` via `workspace:*`. Dogfood target — `pnpm dev:demo`. |

Shared config: `biome.json`, `turbo.json`, `pnpm-workspace.yaml`, `vitest.config.ts`, `tsconfig` per package.

## Workflow

```bash
pnpm dev          # turbo: runs the demo against local core
pnpm build        # build all packages
pnpm typecheck    # tsc across the graph
pnpm check        # biome (format + lint + organize imports)
pnpm check:fix    # auto-fix what biome can
pnpm test         # vitest
pnpm test:e2e     # playwright (builds core first, boots the e2e fixture project)
```

Filter to one package: `pnpm core <script>` / `pnpm cli <script>` / `pnpm mcp <script>`.

Releases go through changesets: `pnpm changeset` on any PR touching `packages/*`, then CI opens the release PR and publishes on merge. Never bump versions or edit `CHANGELOG.md` by hand.

**After changing `packages/core/src`, rebuild it (`pnpm core build`) before testing the demo** — documents import the published `dist` bundle, not the source.

## Architecture notes

- **Two copies of core exist at runtime.** The viewer imports `src/app/**`; a document imports the built `dist` bundle via `@open-document/core`. Anything that must be *shared* between them (React context, the outline store) is stashed on `globalThis` — see `src/app/lib/page-context.tsx` and `src/app/lib/outline.ts`. A new shared singleton must follow the same pattern or it will silently split in two.
- **Documents are discovered through a virtual module.** `src/vite/open-doc-plugin.ts` globs `docs/*/index.{tsx,jsx,ts,js}` and generates `virtual:open-doc/docs`, plus a cache-bust token per doc for hot reload.
- **The outline is a DOM scan, not a parse.** `collectOutline()` walks rendered page frames for headings. The viewer scans after fonts settle; both exporters scan their own offscreen copy before serializing, then restore the previous snapshot.
- **Two kinds of page entries.** `DocModule.default` is `DocEntry[]`: a component is one fixed sheet, a `flow()` section is continuous content the framework paginates. `lib/flow.ts` holds the pure packer (`paginateBlocks`, unit-tested), `lib/flow-measure.ts` does the offscreen DOM measurement, `lib/use-doc-pages.ts` joins them into the rendered page list that the viewer, the thumbnails, and both exporters all consume. Anything that used to read `doc.default` directly must go through `useDocPages`.
- **Page geometry is one function.** `resolvePageGeometry(meta)` owns the CSS-pixel size *and* the `@page` descriptor. Never hardcode 794 × 1123 anywhere else.
- **Document operations live in `src/ops/`, not in the routes.** `routes/docs.ts` and the MCP tools both call the same functions, so a conflict check or a validation rule is written once. An `OpsError` carries the HTTP status the transport should report. Anything new that mutates a document belongs there, not inline in a route.
- **`@open-document/mcp` is imported dynamically and is not a core dependency.** `mcp-plugin.ts` resolves it through a variable specifier — core must not take a build-time dependency on a package whose peer is core. A missing install warns and disables the endpoint; it is never fatal.
- **Dev-only endpoints live behind `apply: 'serve'`.** `api-plugin.ts` mounts `/__assets/*` (routes under `vite/routes/`), `design-plugin.ts` mounts `/__design`. Every mutating handler calls `validateMutationRequest` first — these write to the user's disk. Path safety for assets is centralized in `files/assets.ts`; never join a user-supplied name onto a directory by hand.
- **The inspector edits source, not the DOM.** `loc-tags-plugin.ts` stamps `data-od-loc="line:col"` onto host JSX in document sources (dev only); the overlay reads that attribute, and `/__edit/*` (routes/edit.ts) applies the change through `editing/edit-ops.ts` (single text child only — anything else is refused) or writes a `@doc-comment` marker via `editing/comments.ts`. Markers are base64url JSON so a note can hold quotes and newlines.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [simonliu-ai-product/open-doc](https://github.com/simonliu-ai-product/open-doc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-18 -->
