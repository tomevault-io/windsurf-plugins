---
trigger: always_on
description: This file provides guidance when working with code in this repository.
---

# CLAUDE.md

This file provides guidance when working with code in this repository.

## Overview

`angular-cli-ghpages` is an Angular CLI builder/schematic that deploys Angular applications to GitHub Pages, Cloudflare Pages, or any Git repository. It wraps the `gh-pages` npm package and integrates with Angular CLI's deployment infrastructure via `ng deploy`.

## Development Commands

All development commands must be run from the `src` directory:

```bash
cd src
```

**IMPORTANT:** The `src/.npmrc` file contains `ignore-scripts=false` to override global npm settings. **DO NOT DELETE OR MODIFY this file** - it's required for build scripts to run.

### Build
```bash
npm run build
```
Build process: `prebuild` (clean + regenerate schema.d.ts) → `build` (tsc) → `postbuild` (copy metadata to dist/).

`schema.json` is source of truth for deployment options. Editing it requires rebuild.

### Test
```bash
npm test
```

### Local Development with npm link

For testing changes locally with an Angular project:

1. Build and link from `src/dist`:
   ```bash
   cd src
   npm run build
   cd dist
   npm link
   ```

2. In your Angular test project:
   ```bash
   npm link angular-cli-ghpages
   ng add angular-cli-ghpages
   ng deploy --dry-run  # Test without deploying
   ```

### Debugging

To debug the deploy builder in VSCode, use this `launch.json` configuration in your Angular project:

```json
{
  "type": "node",
  "request": "launch",
  "name": "Debug ng deploy",
  "skipFiles": ["<node_internals>/**"],
  "program": "${workspaceFolder}/node_modules/@angular/cli/bin/ng",
  "cwd": "${workspaceFolder}",
  "sourceMaps": true,
  "args": ["deploy", "--no-build"]
}
```

Alternatively, debug from command line:
```bash
node --inspect-brk ./node_modules/@angular/cli/bin/ng deploy
```

For debugging the standalone engine directly, use the "Launch Standalone Program" task in VSCode (configured in `.vscode/launch.json`).

### Publishing

Publishing uses [npm Trusted Publishers](https://docs.npmjs.com/trusted-publishers) with OIDC – no tokens stored in CI!

1. Go to **Actions** → **Publish to npm**
2. Click **Run workflow** → select branch
3. Leave "Dry-run" checked to test, or uncheck for real publish
4. Wait for approval (5 min timer + required reviewer)

Publishes with provenance attestation for supply chain security.

For pre-release versions, after publishing:
```bash
npm dist-tag add angular-cli-ghpages@X.X.X-rc.X next
```

## Architecture

### Entry Points

1. **Angular CLI Integration** (`src/deploy/`):
   - `builder.ts` - Angular builder entry point, called by `ng deploy`
   - `actions.ts` - Orchestrates build and deployment process
   - `schema.json` - Defines CLI options/arguments

2. **Schematic** (`src/ng-add.ts`):
   - Implements `ng add angular-cli-ghpages`
   - Adds deploy target to `angular.json`

3. **Standalone CLI** (`src/angular-cli-ghpages`):
   - Bash script for non-Angular CLI usage
   - Uses `commander` for CLI parsing

4. **Core Engine** (`src/engine/`):
   - `engine.ts` - Core deployment logic (wraps gh-pages)
   - `defaults.ts` - Default configuration values

### Deployment Flow

```
ng deploy
  ↓
builder.ts (createBuilder)
  ↓
actions.ts (deploy function)
  ├─→ Build Angular app (if not --no-build)
  │   Uses BuilderContext.scheduleTarget()
  └─→ engine.run()
      ├─→ Prepare options (tokens, CI env vars)
      ├─→ Create .nojekyll file (bypasses Jekyll on GitHub)
      ├─→ Create 404.html (copy of index.html for SPAs)
      ├─→ Create CNAME file (if custom domain)
      └─→ Publish via gh-pages package
```

### Build Target Resolution

**Precedence:**
1. `prerenderTarget` - For SSG/prerendering builds (if specified, overrides all others)
2. `buildTarget` - Standard build target (if specified)
3. Default - `${project}:build:production`

**Implementation details:**
- Static build target: `buildTarget || default` (see `src/deploy/builder.ts`)
- Final target: `prerenderTarget || staticBuildTarget` (see `src/deploy/builder.ts`)

Output directory resolution:
- Checks `angular.json` for `outputPath`
- If string: appends `/browser` (modern Angular convention)
- If object: uses `${base}/${browser}` properties
- Can be overridden with `--dir` option

### Token Injection

The engine automatically injects authentication tokens into HTTPS repository URLs:

1. Discovers remote URL from current git repo (if `--repo` not specified)
2. Checks environment variables in order: `GH_TOKEN`, `PERSONAL_TOKEN`, `GITHUB_TOKEN`
3. Transforms: `https://github.com/...` → `https://x-access-token:TOKEN@github.com/...`

**Note:** Tokens only work with HTTPS, not SSH URLs (git@github.com).

### CI Environment Detection

The engine appends CI metadata to commit messages when running on:
- Travis CI (`TRAVIS` env var)
- CircleCI (`CIRCLECI` env var)
- GitHub Actions (`GITHUB_ACTIONS` env var)

### Option Name Mapping

**CRITICAL:** Angular CLI passes `--no-X` flags as `noX: true`, NOT as `X: false`. The engine must manually invert these:

- `--no-dotfiles` → Angular passes `{ noDotfiles: true }` → Engine converts to `{ dotfiles: false }`
- `--no-notfound` → Angular passes `{ noNotfound: true }` → Engine converts to `{ notfound: false }`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [angular-schule/angular-cli-ghpages](https://github.com/angular-schule/angular-cli-ghpages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
