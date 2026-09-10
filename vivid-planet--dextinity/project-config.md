---
trigger: always_on
description: - you can expect setup is already done: all dependencies are installed and builds are done
---

# Agent Instructions

## Local Usage (IDE / CLI)

- you can expect setup is already done: all dependencies are installed and builds are done
- when making changes, affected packages need to be built, except the build watcher process is running (managed by dev-pm)
- services can be started by dev-pm

## Cloud / Web Usage (Fresh Clone)

When running in the cloud/web environment and repo is cloned from scratch, if required you can:

1. Install dependencies (**only** run if not yet done by eg. pipeline setup, check if `node_modules` exists)
    - `NODE_USE_ENV_PROXY=1 ./install.sh`
2. Run build for packages you are working on (**only** run if build file doesn't exist yet, check if `lib` exists in package)
    - `pnpm --recursive --filter '<package-name>' run build`

**If Docker is available:**

- You can start the full demo application using dev-pm and access it at localhost. Do **not** use docker compose directly, always use dev-pm and start `demo-docker`.

**If Docker is not available:**

- You cannot start the full demo application

When making changes to demo api, either start the api and verify it's running or at least run `pnpm run console --help` to verify AppModule is configured correctly and schema.gql/block-meta.json regenerates.

## Linting

Run the appropriate checks after every change and fix all reported errors.

### Post-Change Workflow

After making changes to any package (`packages/*`) or the demo project (`demo/*`), run `lint:fix` to auto-fix ESLint and Prettier issues:

```bash
# Fix a single package (from the package folder or repo root)
pnpm run lint:fix
# or from repo root:
pnpm --filter <package-name> run lint:fix

# Fix all packages at once (from repo root)
pnpm run lint:fix
```

`tsc` errors must be fixed manually — `lint:fix` does not cover type errors.

### `lint:fix` scripts

Every package and demo service exposes these scripts:

- `lint:fix` — runs `lint:fix:eslint` and `lint:fix:prettier` in parallel (plus `lint:fix:style` in `demo/site`)
- `lint:fix:eslint` — runs ESLint with `--fix` (also applies Prettier formatting via `eslint-plugin-prettier`)
- `lint:fix:prettier` — runs Prettier with `--write` for any remaining formatting issues
- `lint:fix:style` — runs Stylelint with `--fix` (`demo/site` only)

The root-level `lint:fix` runs all workspace `lint:fix` scripts recursively, then formats root-level config files.

### In `docs/`

Run from the `docs/` folder:

```bash
# Auto-fix ESLint + Prettier issues
pnpm run lint:fix

# Verify all checks pass: prettier, eslint
pnpm run lint
```

## Demo application

This repository includes a demo application which is showcasing the libraries.

**Api:**
Demo backend API (NestJS, PostgreSQL)

- Start using `pnpm exec dev-pm start @demo-api`
- access at: http://localhost:4000/

**Admin:**
Demo admin app (Vite, React, Apollo)

- Start using `pnpm exec dev-pm start @demo-admin`
- access at: http://localhost:8000/
- Usually Login as "Admin" in the IDP Login Page

**Site:**
Demo frontend site (Next.js)

- Start using `pnpm exec dev-pm start @demo-site`
- access at: http://localhost:3000/

### dev-pm

Use dev-pm for managing demo processes:

- command: `pnpm exec -- dev-pm`
- config: `dev-pm.config.ts`

Common commands:

- start script/service: `pnpm exec -- dev-pm start scriptName`
- start group `pnpm exec -- dev-pm start @group`
- view status `pnpm exec -- dev-pm status`
- view logs `pnpm exec -- dev-pm logs --lines 100 scriptName`

## Packages

### Admin (`packages/admin/`)

| Package                         | Description                                                                                   |
| ------------------------------- | --------------------------------------------------------------------------------------------- |
| `@dextinity/admin`              | Core admin components and framework (React, MUI, Apollo, Final Form)                          |
| `@dextinity/admin-babel-preset` | Shared Babel preset for transpiling admin packages                                            |
| `@dextinity/admin-color-picker` | Color picker form field component                                                             |
| `@dextinity/admin-date-time`    | Date/time picker form field components                                                        |
| `@dextinity/admin-icons`        | SVG icon library as React components                                                          |
| `@dextinity/admin-rte`          | Rich text editor component (Draft.js-based)                                                   |
| `@dextinity/admin-generator`    | CLI tool that generates admin CRUD UIs from GraphQL schemas (`dextinity-admin-generator` bin) |
| `@dextinity/cms-admin`          | Admin UI for CMS features (pages, blocks, content management)                                 |
| `@dextinity/brevo-admin`        | Admin UI for Brevo email/marketing integration                                                |

### API (`packages/api/`)

| Package                    | Description                                                                                |
| -------------------------- | ------------------------------------------------------------------------------------------ |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vivid-planet/dextinity](https://github.com/vivid-planet/dextinity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
