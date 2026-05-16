---
trigger: always_on
description: Daggerheart Tools is an Obsidian plugin (TypeScript + Svelte) that provides a GM toolkit for the Daggerheart RPG system. It renders YAML code blocks inside Obsidian notes as interactive stat blocks for adversaries and environments, and provides modals for creating/editing/searching game content stored in plugin settings.
---

# Daggerheart Tools — Claude Context

## Project Overview

Daggerheart Tools is an Obsidian plugin (TypeScript + Svelte) that provides a GM toolkit for the Daggerheart RPG system. It renders YAML code blocks inside Obsidian notes as interactive stat blocks for adversaries and environments, and provides modals for creating/editing/searching game content stored in plugin settings.

**Repo:** https://github.com/gotsanity/obsidian-daggerheart-tools
**Current version:** 1.1.1

---

## Branch & Deployment Pipeline

```
feature-branch → development → master
```

- **Feature branches** are where individual fixes and features are developed (e.g. `fix-complex-statblocks`). Always branch from `development`.
- **`development`** is the integration branch — PRs from feature branches merge here first.
- **`master`** is the release branch (the repo default). Merging to `master` triggers the automated release pipeline.

### Development Workflow (step-by-step)

This is the process to follow for every feature or fix:

1. **Create a feature branch from `development`**
   ```bash
   git checkout development
   git pull origin development
   git checkout -b feat/my-feature-name
   ```

2. **Implement the feature** — only make changes relevant to the feature being worked on. Do not mix unrelated fixes or refactors into the same branch.

3. **Verify it works** — run `npm run build` to confirm no type errors or build failures before committing.

4. **Commit using Conventional Commits**
   ```bash
   git add <specific files>
   git commit -m "feat: add per-item export selection"
   ```
   Common prefixes: `feat:`, `fix:`, `docs:`, `refactor:`, `chore:`. These drive the CHANGELOG and version bump — write subject lines as customer-facing descriptions.

5. **Push and sync with origin**
   ```bash
   git push -u origin feat/my-feature-name
   ```

6. **Open a merge request (PR) targeting `development`** via GitHub. Merge once reviewed/verified.

7. **If the work addresses a tracked GitHub issue**, comment on the issue with a summary of what was changed and how it resolves the problem. Close the issue once the feature branch has been merged into `development` and the fix is fully integrated — do not close prematurely while the PR is still open or the feature is incomplete.

8. **After the PR is merged, return to `development` and sync**
   ```bash
   git checkout development
   git pull origin development
   ```
   Always do this before starting a new feature to avoid accidentally committing new work onto a stale feature branch.

9. **When all features for a release are in `development`**, open a PR from `development` → `master` and merge it.

8. **release-please runs automatically** on push to `master`, opening or updating a release PR. **Merging the release PR requires manual action by the repo owner** — it triggers the GitHub Release, artifact upload, and npm publish.

### Automated Release (release-please)

`.github/workflows/release-please.yml` runs on every push to `main` (and can be triggered manually).

**How it works:**

1. **Job: `release-please`** — Uses `google-github-actions/release-please-action@v3` in `manifest` mode with `release-type: node`. This action reads conventional commits merged into `main` and automatically:
   - Maintains a release PR that accumulates changes and bumps `package.json` version
   - When the release PR is merged, creates a GitHub Release and git tag

2. **Job: `upload-build`** — Runs only when `release_created == true` (i.e., the release PR was just merged). It:
   - Checks out code and sets up Node 20
   - Runs `npm install && npm run build`
   - Packages build artifacts into `Daggerheart-Tools/` directory
   - Creates `Daggerheart-Tools.zip`
   - Uploads four release assets to the GitHub Release:
     - `Daggerheart-Tools-{tag}.zip` — full plugin zip
     - `main.js` — compiled bundle
     - `manifest.json` — Obsidian plugin manifest
     - `styles.css` — compiled styles

### npm Publish

`.github/workflows/npm-publish.yml` triggers on release creation (or manually). It publishes the package to npm under `@gotsanity/obsidian-daggerheart-tools` with public access, using `secrets.npm_token`.

### Summary: What triggers a release

| Action | Effect |
|--------|--------|
| Push feature branch | Nothing automated |
| Merge to `development` | Nothing automated |
| Merge to `master` | release-please opens/updates a release PR |
| Merge the release PR (manual) | GitHub Release created + build artifacts uploaded + npm publish |

> **Do not manually bump versions or create tags.** release-please manages `package.json` version, `CHANGELOG.md`, and git tags based on conventional commit messages.

---

## Build System

- **Dev:** `npm run dev` — watches with esbuild, inline sourcemaps, no type-check
- **Prod:** `npm run build` — `tsc -noEmit` type-check, then esbuild minified bundle
- **Outputs:** `main.js` (bundled TS + Svelte) and `styles.css` (compiled SASS)
- **Bundler:** esbuild with `esbuild-svelte` and `esbuild-sass-plugin`

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gotsanity/Obsidian-Daggerheart-Tools](https://github.com/gotsanity/Obsidian-Daggerheart-Tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
