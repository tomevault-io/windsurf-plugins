---
trigger: always_on
description: Monorepo with npm workspaces + turbo, 4 packages under `packages/*`. Build order is managed by `tsc -b` via project references.
---

# AGENTS.md - OpenCode Trace

## Project Structure

Monorepo with npm workspaces + turbo, 4 packages under `packages/*`. Build order is managed by `tsc -b` via project references.

| Package | Purpose | Key Dependencies | Exports |
|---------|---------|------------------|---------|
| `@opencode-trace/core` | Core (parse, store, query, record, state, format, transform) | zod, winston | `.` + `./state` subpath |
| `@opencode-trace/cli` | CLI tool (`opencode-trace`) | core | `bin: opencode-trace` |
| `@opencode-trace/plugin` | OpenCode plugin (intercepts fetch) | @opencode-ai/plugin, core | plugin entry |
| `@opencode-trace/viewer` | Web viewer (`opencode-trace-viewer`) | fastify, vue, core | `bin: opencode-trace-viewer` |

Root `.opencode/opencode.json` loads `@opencode-trace/plugin`.

## Commands

```bash
npm run build           # turbo run build (dependsOn: ^build — core first)
npm run build:frontend  # turbo run build:frontend — Vite build for viewer Vue app
npm run test            # turbo run test (dependsOn: build — always builds first)
npm run clean           # turbo run clean (rm -rf dist *.tsbuildinfo)
npm run dev             # concurrently "tsc -b --watch" "nodemon viewer"
npm run cli             # node packages/cli/dist/index.js
npm run viewer          # node packages/viewer/dist/cli.js
npx tsc --noEmit        # type-check all packages (CI lint job)
```

## Testing

- Vitest, tests colocated: `packages/*/src/**/*.test.ts`
- Root vitest.config uses jsdom env (for Vue components) and excludes `dist/` + `src/viewer/frontend/`
- `packages/core/vitest.config.ts` sets `globals: true`
- Viewer uses `@vue/test-utils` for component tests
- Tests always run after build (turbo `dependsOn: ["build"]`)

## Build Quirks

- **Clean always** after stale builds: `rm -rf dist *.tsbuildinfo` — tsbuildinfo files cause `tsc -b` to skip rebuilding
- **Viewer build is two-phase**: `tsc` (server) → `npm run build:frontend` (Vite/vue → `dist/public/`)
- **Viewer tsconfig** excludes `src/frontend/` — the Vue app has its own Vite build
- No eslint, prettier, or formatter config exists in this repo.

## Release Process

Monorepo versioning is managed by [Changesets](https://github.com/changesets/changesets).
**Do NOT manually edit `version` fields in any `packages/*/package.json`** — Changesets owns them.

### Developer Workflow

1. After making code changes, declare the change:

   ```bash
   npx changeset
   ```

   Interactive prompt: pick affected packages (space-separated) → bump level (major/minor/patch) → changelog message.
   This creates `.changeset/<random-name>.md` — **commit this file with the PR**.

2. Open a PR to `main`. CI runs normal build + tests.

3. On merge, `.github/workflows/changesets.yml` detects pending changesets and automatically
   opens/updates a **`Version Packages`** PR containing the version bumps, internal dependency
   updates, and `CHANGELOG.md` entries for each affected package.

4. Merge the `Version Packages` PR → the same workflow publishes to npm
   (in dependency order: `core` → `cli` → `plugin` → `viewer`, skipping already-published versions)
   and creates a GitHub Release with provenance.

### Internal Dependency Sync

`config.json` has `updateInternalDependencies: "patch"`. When `core` is bumped, the
`"@opencode-trace/core": "x.y.z"` reference in `cli` / `plugin` / `viewer`
is auto-bumped in the `Version Packages` PR. No manual cross-package edits.

### Local Manual Escape Hatch

If CI is unavailable, you can do the whole flow locally:

```bash
npm run version      # bump versions + sync internal deps + generate CHANGELOG
npm run release      # build + publish to npm (requires NPM_TOKEN env)
```

### Required GitHub Secrets

- `NPM_TOKEN` — npm automation token (publishing)
- `GITHUB_TOKEN` — auto-provided, used for the `Version Packages` PR

## Windows CI Compatibility (IMPORTANT)

This project runs CI on both Linux and Windows. File system behavior differs significantly:

### MUST Follow
- **`fs.rename()` is NOT atomic on Windows (NTFS)**. Unlike POSIX, Windows rename can fail
  with `EACCES`/`EPERM` if the destination is locked (antivirus, delayed flush). All rename
  operations in this project use `safeRename()` with retry logic (3 retries, exponential backoff).
  NEVER use bare `fs.rename()` for `.tmp → final` atomic write patterns — always use `safeRename()`.
- **Test file polling MUST match assertion filters**. `waitForFiles()` and similar helpers must
  use the SAME file filter regex as the assertion (e.g. `/^\d+\.json$/`). Using `.endsWith(".json")`
  counts `metadata.json` and can cause early return with fewer record files than expected, especially
  on slow Windows CI where writes take longer.
- **Always flush before assertions**. After async writes, call `plugin.flush()` (or equivalent)
  before reading the filesystem. Windows I/O is slower and NTFS metadata caching means `readdirSync`
  may not immediately reflect completed writes.
- **Test timeouts on Windows need margin**. The default `waitForFiles` timeout (5s) is sufficient
  but tight on Windows CI. If tests become flaky, increase rather than decrease.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aixmoyu/opencode-trace](https://github.com/aixmoyu/opencode-trace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
