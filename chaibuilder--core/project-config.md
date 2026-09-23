---
trigger: always_on
description: `chaicore` — the open-source ChaiBuilder package: visual editor, block registry,
---

# Agents — `chaicore`

## What this package is

`chaicore` — the open-source ChaiBuilder package: visual editor, block registry,
RSC renderer, server config/actions and DB adapters. Published as a standalone
npm package **and** vendored into host apps as a git subtree (one-way: hosts
pull, and contribute back by PR here). Requires Next.js ≥ 15.3, React ≥ 19,
Tailwind 4.

**It must build and test standalone.** Nothing here may depend on the host app it
happens to be vendored into.

## Two editions, one `src/`

`chaicore` and `chaipro` (the commercial edition, core plus plugins) share one
`src/` tree. Every file under `src/` is byte-identical in both repos except the
paths in `src-sync.exclude` — `src/edition/` (each repo's own copy: package
identity, always-on plugins, plugin barrels, test harness) and the pro-only trees
(`src/payload/` and the pro plugin directories). Changes travel between the repos
as one `chore(sync): …` commit made by `pnpm sync:src`; `pnpm sync:check` proves
the trees match. Read `SYNC.md` before touching anything that names the package,
a plugin, or the edition: runtime strings use `CHAI_PACKAGE_NAME` from
`~/edition/identity`, comments and JSDoc write subpaths as `<pkg>/…`, and shared
code never imports a plugin.

## Layout (`src/`)

- `registry/` — block registration API (`registerChaiBlockProps`, `stylesProp`,
  `builderProp`, pipes, `v2/runtime`). The public surface custom blocks import.
- `web-blocks/` — built-in blocks (box, heading, image, text, rte, list, form, …).
- `builder/` — the editor UI (largest tree, ~640 files).
- `render/` — RSC/SSR renderer: `block-renderer`, `blocks-renderer`,
  `binding-engine`, `binding-pipes`, `apply-binding`, design tokens, `rsc/`, `async/`.
- `server/` — server runtime: `build-config`, `create-chai-builder`,
  `get-chaibuilder`, `chai-actions/`, `builder-actions/`, `plugin-api/`, `rbac/`,
  `repeater-data/`, `only-server.ts` (browser guard).
- `plugins/` — one directory per feature (`empty-page-starter`, `page-errors`).
  Each has `client/`, optionally `server/`, `schema/`, `permissions.ts`.
  `plugins/client.ts` and `plugins/server.ts` are shells that forward the
  edition's barrels (`src/edition/*-plugins-barrel.ts`). Nothing registers
  automatically — the host names the plugins it wants.
- `edition/` — the only directory both editions have and keep different (never
  synced): identity, always-on plugin lists, plugin barrels, integration-test
  schema and harness. See `src/edition/README.md`. The rest of the exceptions in
  `src-sync.exclude` exist only in `chaipro`: `src/payload/` and its plugins.
- `db/` — adapters: libsql, d1, better-sqlite3.
- `drizzle/` — core schema/relations (`schema.sqlite.ts`, `relations.sqlite.ts`)
  plus `seed/`. The Postgres twin of the core schema lives in the pro edition only.
- `tailwind/` — runtime page-CSS compiler (`v4.ts`). Tailwind v4 only.
- `nextjs/` — `withChaiBuilder`, server/render/render-client entries.
- `ai/` — statically-imported provider adapters (`openrouter`, `openai-compatible`).
- `components/ui/` — the shared shadcn/Radix + `class-variance-authority` primitives.
- `types/`, `utils/`, `constants/`, `lib/`, `theme/`.
- `tests/` — shared test infrastructure only: `setup/` (db, migrations, seeding,
  transaction manager), `mocks/`, `utils/` (factories, assertions). Actual specs
  live beside their sources.

## API reference

`.agents/skills/chaibuilder/AGENTS.md` is the full extension-API document —
custom blocks, block props schemas, sidebar panels, slots, lifecycle hooks,
feature flags, server config, data providers, rendering. It is the reference for
anything consumer-facing; do not restate or contradict it here. Other skills:
`.agents/skills/testing-patterns/`, `custom-hooks-pattern/`,
`vercel-react-best-practices/`, `web-design-guidelines/`,
`anthropic-frontend-design/`.

## Setup and commands

Run from the repo root:

```bash
pnpm build              # tsup (esm+cjs) then scripts/build-dts.mjs
pnpm dev                # tsup --watch
pnpm typecheck          # tsc --noEmit — the real typecheck; the root has none
pnpm lint               # eslint src/
pnpm format             # prettier --write
pnpm test               # vitest --run (unit)
pnpm test:integration   # vitest --run --config vitest.config.integration.ts
pnpm db:test:generate   # drizzle-kit generate for the SQLite test schema
```

The declaration build is split into sequential slices (`CHAI_DTS_GROUPS` /
`CHAI_DTS_GROUP`) because rollup-plugin-dts OOMs holding ~150 entries in one
module graph. Do not collapse it back into a single pass.

## Import boundaries (ESLint-enforced)

Three separate `no-restricted-imports` blocks in `eslint.config.mjs`. Read the
comments there before working around any of them.

1. **Subtree boundary (error)** — no host-app aliases (`@/…`, `~~/…`, `#/…`)
   anywhere in `src/`. This repo's own alias is `~/` → `./src`. Code that reaches
   into the host compiles in the host and breaks when the package is built or
   published alone.
2. **Plugin boundary (error)** — shared core code (everything outside
   `src/plugins`, `src/payload` and `src/edition`) must not import a plugin.
   Dependencies point one way: plugins consume core, never the reverse. Invert

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chaibuilder/core](https://github.com/chaibuilder/core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
