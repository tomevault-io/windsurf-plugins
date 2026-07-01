---
trigger: always_on
description: PodNotes is an Obsidian community plugin for listening to podcasts, tracking
---

# Repository Guidelines

## Project Overview
PodNotes is an Obsidian community plugin for listening to podcasts, tracking
playback progress, creating podcast notes, capturing timestamps, downloading
episodes, using local audio files, and exposing a small API for workflow plugins.

## Project Structure
Source code lives in `src/`. Core plugin registration and lifecycle wiring live
in `src/main.ts`; public API code is under `src/API/`; parsing code is under
`src/parser/`; stores and controllers are under `src/store*`; utility functions
are under `src/utility/`; Svelte UI lives under `src/ui/`; shared types live in
`src/types/`.

Tests are colocated with source files as `*.test.ts` where practical. Shared
test mocks live in `tests/mocks/`. User-facing documentation lives in `docs/`
and is built with MkDocs.

Generated plugin artifacts such as `main.js` and source maps are ignored by git
and should not be hand-edited. Production builds write `main.js` at the repo
root for release packaging; development builds write into `build/` and maintain
root symlinks for local Obsidian loading.

## Tooling
- Use Node 22. The repo has `.nvmrc`, `.npmrc`, and `package.json` engines for
  this.
- Use npm for package management and scripts. Do not introduce another package
  manager unless the migration is intentional and removes the old lockfile.
- Use Conventional Commits (`feat:`, `fix:`, `test:`, `docs:`, `chore:`) so
  semantic-release can determine versions.
- If work resolves a GitHub issue, prefer an issue-linked branch workflow before
  implementation.

## Common Commands
- `npm install`: install dependencies for local development.
- `npm run dev`: watch-mode development build via Vite.
- `npm run typecheck`: run `tsc --noEmit`.
- `npm run lint`: run ESLint against TypeScript sources.
- `npm run format:check`: run the configured Biome check.
- `npm run check:a11y`: run `svelte-check --fail-on-warnings`.
- `npm run test`: run Svelte checks and the Vitest suite.
- `npm run build`: type-check and produce the production plugin bundle.
- `npm run docs:build`: build the MkDocs documentation.
- `npm run docs:deploy`: build docs and deploy `docs/site` to Cloudflare Pages.

Before opening a PR or cutting a release, run the CI-equivalent checks locally:

```bash
npm run lint
npm run format:check
npm run typecheck
npm run build
npm run test
npm run docs:build
```

## Testing
Vitest runs in jsdom and aliases `obsidian` to `tests/mocks/obsidian.ts`.
Prefer unit tests for pure utility, parser, store, API, and component behavior.
Use Testing Library for Svelte component behavior instead of asserting on
implementation details.

When a bug depends on real Obsidian runtime behavior, reproduce it in Obsidian
before changing code and verify it there after the fix. Timestamp links, URI
handling, playback restore, downloaded/local media, file writes, settings
migrations, and workspace/view behavior are runtime-sensitive and should not be
trusted to jsdom alone.

For runtime verification, record the exact Obsidian version, platform, vault
setup, feed or local file used, command or URI invoked, console/runtime errors,
and observed plugin state before and after the action.

## Obsidian Runtime Workflow
Use a dedicated development vault for manual or scripted Obsidian checks. Ensure
the vault's PodNotes plugin folder points at this checkout's generated plugin
artifacts before trusting runtime evidence.

If using the `obsidian` CLI, pass the vault selector consistently and prefer
scripted, repeatable checks for non-trivial flows. For bugs involving commands
or URIs, test both the user-facing path and the direct command/URI path when
possible.

### Shared dev vault (main checkout)
For work in the canonical `/Users/christian/Developer/PodNotes` checkout, use the
shared `dev` vault and target it explicitly with the `obsidian` CLI:

```bash
npm run dev
# reload or re-enable PodNotes in the dev vault, e.g.:
obsidian vault=dev plugin:reload id=podnotes
# trigger the relevant command, UI flow, or obsidian://podnotes URI
obsidian vault=dev eval code='app.plugins.plugins.podnotes?.manifest?.version'
# inspect console/errors and plugin state
```

- Dev vault root: `/Users/christian/Developer/dev_vault/dev`.
- PodNotes plugin folder in the vault:
  `/Users/christian/Developer/dev_vault/dev/.obsidian/plugins/podnotes`, whose
  `main.js`/`manifest.json` symlinks point at the canonical checkout's artifacts.
- Only one checkout can own those symlinks at a time, so the shared `dev` vault
  is for the main checkout. Worktrees must use the isolated wrapper below.

### Isolated worktree vault (parallel worktrees)
In a worktree (e.g. `/Users/christian/orca/workspaces/PodNotes/<slug>`), do **not**
race the shared `dev` vault — multiple worktree agents would clobber each other on
the plugin symlink, `data.json`, and `plugin:reload`. Use the isolated worktree
wrapper instead, which provisions a worktree-local vault under
`.obsidian-e2e-vaults/podnotes-<worktree>` (git-ignored), starts or reuses a
private-`HOME` Obsidian instance bound to that vault, disables Restricted Mode,
waits until PodNotes is live, and then runs your command with the right
`vault=<worktree vault>` and private `HOME` already applied:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chhoumann/PodNotes](https://github.com/chhoumann/PodNotes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
