---
trigger: always_on
description: NiiVue is a browser-based medical image visualization ecosystem. This monorepo contains the core WebGPU/WebGL2 viewer, React bindings, extension libraries, demo applications, and a Python Jupyter widget (`ipyniivue`). The stack is TypeScript, Vite, Bun (package manager + runtime), Biome (lint/format), Nx (task orchestration + caching), and Pixi/hatchling (Python environments + wheels).
---

# AGENTS.md — AI Agent Instructions for NiiVue Monorepo

## Project overview

NiiVue is a browser-based medical image visualization ecosystem. This monorepo contains the core WebGPU/WebGL2 viewer, React bindings, extension libraries, demo applications, and a Python Jupyter widget (`ipyniivue`). The stack is TypeScript, Vite, Bun (package manager + runtime), Biome (lint/format), Nx (task orchestration + caching), and Pixi/hatchling (Python environments + wheels).

See `README.md` for a package overview and `CONTRIBUTING.md` for detailed development docs.

## Workspace layout

```
packages/       # Libraries (publishable). See each package's README for details.
apps/           # Demo applications (private, not published). See each app's README.
nx-tools/       # Custom Nx plugins and scripts (boundary checks, Pixi plugin, release)
```

- Each `packages/*` and `apps/*` directory has its own `README.md`, `package.json`, `project.json`, and `tsconfig.json`.
- When working on a specific package or app, read its README first.

### Tags

Every project has two tags in its `project.json`:

- **`type:app`** or **`type:lib`** — apps live in `apps/`, libs live in `packages/`.
- **`lang:typescript`** or **`lang:python`** — all current projects are `lang:typescript`.

### Workspace links

Dependencies between packages use `"workspace:*"` in `package.json`. Bun resolves these from the workspace root. The root `package.json` declares workspaces:

```json
"workspaces": ["apps/*", "packages/*"]
```

## Commands

**Package manager: Bun.** Always use `bun` and `bunx`, not `npm`/`npx`/`pnpm`/`yarn`.

```bash
bun install                          # Install all dependencies

bunx nx build <project>              # Single project (builds deps first)
bunx nx run-many -t build            # All projects
bunx nx affected -t build            # Only projects affected by current changes

bunx nx test <project>               # Single project
bunx nx affected -t test             # Only affected

bunx nx lint <project>               # Single project (Biome)
bunx nx affected -t lint             # Only affected

bunx nx typecheck <project>          # Single project (tsc --noEmit)
bunx nx affected -t typecheck        # Only affected

bunx nx format <project>             # Auto-fix formatting (Biome)

bunx nx dev <project>                # Start dev server

bun run check-boundaries             # Enforce module boundary rules
bunx nx reset                        # Clear Nx cache
```

Prefer `nx affected` over `nx run-many` when working on a branch.

## Creating new code

### When to create a new lib vs. app

- **Library** (`packages/`): reusable code published or consumed by other workspace projects. Tag `type:lib`.
- **Application** (`apps/`): runnable demo or standalone app, not imported by others. Tag `type:app`.

### Steps to add a new TypeScript project

1. Create directory under `apps/` or `packages/`.
2. Add `package.json` with `"type": "module"`. Use `workspace:*` for internal deps.
3. Add `tsconfig.json` — copy from an existing project (there is no shared `tsconfig.base.json`).
4. Add `project.json` following the pattern of an existing project. Required fields:
   - `name`, `projectType` (`"library"` or `"application"`),
   - `tags` (e.g. `["type:lib", "lang:typescript"]`),
   - `targets` — at minimum `build`, `lint`, `format`, `typecheck`
5. Run `bun install` from the repo root.

This repo does **not** use Nx generators. Projects are created manually.

## Module boundaries

Enforced by `nx-tools/check-boundaries.js`. Run with `bun run check-boundaries`.

1. **No project may depend on an app.** Only libs (`type:lib`) are allowed as dependencies.
2. **Libs cannot depend on apps.**
3. **No cross-language dependencies.** TypeScript ↔ Python imports are forbidden.

## Code style

### Biome (sole linter/formatter)

Configured in root `biome.json`. Applies to all projects.

| Rule | Setting |
|------|---------|
| Indent | 2 spaces |
| Quotes | Single quotes |
| Semicolons | As needed (no mandatory semicolons) |
| `noExplicitAny` | **Error** — do not use `any` |
| `noNonNullAssertion` | **Error** — do not use `!` postfix |
| `noBarrelFile` | **Error** — do not create barrel/index re-export files |
| `noDoubleEquals` | **Error** — use `===` not `==` |
| `noUnusedVariables` | **Error** — prefix intentionally unused vars with `_` |
| `noUnusedImports` | **Error** — remove unused imports |
| `useImportType` | **Error** — use `import type` for type-only imports |
| Import sorting | Enabled via `organizeImports` assist |

**No emoji in source, scripts, or generated reports.** This includes status icons (traffic lights, check marks) in CI output, markdown summaries, and log messages. Use plain text.

### TypeScript

- **Target:** ESNext, **strict mode** enabled everywhere.
- **Path alias:** `@/*` → `./src/*` (used in some packages — check the project's `tsconfig.json`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [niivue/mono](https://github.com/niivue/mono) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
