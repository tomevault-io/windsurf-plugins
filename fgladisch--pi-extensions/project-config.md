---
trigger: always_on
description: npm install                    # Install dev dependencies (run once after clone, after upgrades,
---

# AGENTS.md

## Commands

```bash
# Setup
npm install                    # Install dev dependencies (run once after clone, after upgrades,
                               # or whenever VSCode stops resolving pi types).
                               # Also installs the husky pre-commit hook via the `prepare` script.

# Development
npm run typecheck              # tsc -p tsconfig.typecheck.json --noEmit
npm test                       # jest
npm run test:coverage          # jest with coverage report (HTML in coverage/)
npm run lint                   # eslint --fix over packages/*/{extensions,tests}
npm run lint:file <path>       # eslint --fix on a single file
npm run format                 # prettier --write over **/*.{ts,js,cjs,md,json,yml,yaml}
npm run format:file <path>     # prettier --write on a single file
npm run organize-imports       # organize-imports-cli over all tracked .ts files (via git ls-files)
npm run organize-imports:file <path>  # organize-imports-cli on a single file
npm run changeset              # create a changeset entry for changed package(s)
npm run changeset:version      # apply changesets and bump versions
npm run changeset:publish      # publish packages from changesets
```

There is no build step. Pi loads `.ts` extension files directly.

## Code Conventions

**Mandatory before code changes:** read `./docs/code-conventions.md` before touching implementation code.

Do not rely on memory of conventions from earlier sessions. Re-read the file in the current session, then implement.

## Documentation

When making changes, always update relevant `README.md` files in the same change so user-facing docs stay in sync with behavior and configuration.

## Release Workflow (Changesets + CI + npm Trusted Publisher)

Releases are CI-driven from `.github/workflows/release.yml` on pushes to `main`. npm authentication uses Trusted Publisher/OIDC, so no npm automation token is required.

1. Add a changeset (`npm run changeset`) for every publishable package change.
2. Always update `packages/<package>/CHANGELOG.md` before releasing a new version.
3. Commit and push to `main`.
4. CI runs `lint`, `typecheck`, `test`, then `changesets/action`.
5. `changesets/action` opens/updates a release PR (`chore: release packages`).
6. Merge that release PR to trigger publish with `npm run changeset:publish` through Trusted Publisher.

### Required GitHub and npm settings

- Actions workflow permissions: **Read and write permissions**.
- Enable: **Allow GitHub Actions to create and approve pull requests**.
- Release workflow permissions include `id-token: write`.
- Each npm package is configured with Trusted Publisher for this GitHub repository and `.github/workflows/release.yml`.

### Release caveats

- Do not manually bump package versions for normal releases; let changesets own versioning.
- Do not add an `NPM_TOKEN` secret for publishing; Trusted Publisher handles npm authentication.
- If CI fails with `ENOENT .../packages/<pkg>/CHANGELOG.md`, add `CHANGELOG.md` in that package.

**Pre-commit hook** (`.husky/pre-commit`) runs `npx lint-staged && npm run typecheck && npm test`. For staged `.ts` files lint-staged runs `organize-imports-cli` (sorts/removes unused imports via the TS language service) and then `eslint --fix`; for staged `.{ts,js,cjs,md,json}` files it runs `prettier --write` (see `lint-staged` config in [package.json](package.json)).

## Tooling Preference

- Prefer shell commands (`rg`, `find`, `ls`, `gh`, `jq`) for repository inspection, data extraction, and automation.
- Avoid Python scripts when a bash command can do the job clearly.
- Use Python only when bash would be significantly more complex or less readable.

## Project Layout

```
.
├── packages/
│   ├── pi-<feature>/
│   │   ├── extensions/
│   │   │   ├── index.ts              # Extension entrypoint (default export)
│   │   │   ├── models/
│   │   │   │   ├── index.ts          # Barrel for package-local models/enums
│   │   │   │   ├── *.model.ts        # Package-local type aliases/models
│   │   │   │   └── *.enum.ts         # Package-local enums
│   │   │   └── utils.ts              # Package-local helpers (or utils/*)
│   │   ├── tests/
│   │   │   └── <feature>.spec.ts     # Jest specs colocated with package
│   │   ├── package.json              # Publishable npm package metadata
│   │   └── tsconfig.json             # Package TS project (composite)
├── .changeset/                       # Changesets config + release metadata
├── .github/workflows/release.yml     # Changesets release workflow
├── .husky/pre-commit                 # lint-staged + typecheck + tests
├── .eslintrc.js                      # ESLint (legacy config) + @typescript-eslint + prettier integration
├── jest.config.cjs                   # Workspace Jest config
├── tsconfig.base.json                # Shared TS compiler options
├── tsconfig.json                     # Root project references
├── package.json                      # Workspace tooling + npm workspaces config
└── README.md                         # Workspace overview
```

> **Do not** add a `pi.extensions` field to root `package.json` — that would override pi's auto-discovery and silently disable extensions not listed there.

## Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fgladisch/pi-extensions](https://github.com/fgladisch/pi-extensions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
