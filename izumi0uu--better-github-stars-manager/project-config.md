---
trigger: always_on
description: Keep this file short and practical. Add rules here only when they are core to the repo or fix a repeated mistake.
---

# GitHub Stars Manager Agent Guide

Keep this file short and practical. Add rules here only when they are core to the repo or fix a repeated mistake.

## Scope

- This repo is a Chrome extension that augments the GitHub Stars page with local search, filters, sorting, tags, notes, and sync helpers.
- Prefer repo-specific facts over generic browser-extension assumptions.
- If an API behavior is unclear or recently changed, check official docs before editing code.

## Branch Policy

- `master` is the sole long-lived development and release branch.
- Feature branches, including `feat/firefox`, are temporary integration branches and must merge back into `master`; do not treat them as future development lines.

## Key Paths

- `src/storage/db.ts`: IndexedDB schema and version bumps.
- `src/types/index.ts`: shared domain types and config shape.
- `src/auth/auth-store.ts`: `chrome.storage.local` config normalization.
- `src/api/github-star-source.ts`: GitHub REST/GraphQL sync logic.
- `src/upgrades/backfill-state.ts` and `src/upgrades/tasks.ts`: one-shot feature/data backfills.
- `src/background/index.ts`: backfill orchestration and sync entrypoints.
- `tests/unit`, `tests/integration`, `tests/regressions`, `tests/runtime`: test layers.

## Contributor Workflow

- Before implementation or debugging, consult `CONTRIBUTING.en.md` (or `CONTRIBUTING.md`) for the current build and verification commands.
- Cubby Agent work must also consult `docs/en/cubby-agent.md` (or `docs/zh/cubby-agent.md`) and use the documented diagnostics entry points. Development diagnostics builds are never release evidence.

## Source Of Truth

- IndexedDB is the source of truth for bulk repo data and annotations: `stars`, `tags`, `tagMeta`.
- `chrome.storage.local` is only for lightweight config and UI state: token metadata, locale, theme, onboarding, sync progress, backfill state, and user preferences.
- GitHub is the source of truth for repository metadata such as `archived`, `fork`, `pushed_at`, `created_at`, and `starred_at`.
- Do not infer remote repo state in the UI when the sync layer can persist the canonical field.

## Data Rules

- UI-only behavior changes do not need a storage upgrade.
- Unless `package.json` version has already changed in the current worktree, treat new feature work as unreleased. Do not add compatibility code for hypothetical previously shipped users unless the user explicitly says the behavior has already been released.
- Local dev builds and feature-branch experiments are not releases. If an unreleased migration, backfill, or schema change is revised before shipping, edit the existing unreleased upgrade/backfill in place instead of inventing a new version/id just to support local development data.
- A new lightweight preference in `Config` should be added with a safe default and normalized on read. This usually does not need a DB bump.
- A new persisted field on `Star`, `Tag`, or `TagMeta` requires:
  - updating `src/types/index.ts`
  - bumping Dexie schema in `src/storage/db.ts` if the stored shape changes
  - keeping legacy-row compatibility, usually by treating old `undefined` values as missing
- New remote-derived metadata for existing rows should usually use a feature/data backfill, not an app-version migration.
- Backfills are keyed by capability, not extension version. Once a one-shot backfill is done, it should stay done unless the task definition itself changes.
- Do not run a full sync on every extension update. Full sync is for data completeness gaps that incremental sync or lazy hydration cannot close reliably.

## Upgrade Decision Rules

- Use a Dexie version bump when stored IndexedDB shape or indexes change.
- Use config normalization when only `chrome.storage.local` shape changes.
- Add a backfill task when old local rows are missing data required by a new feature.
- Prefer lazy remote hydration when missing data can be filled gradually without blocking correctness.
- Prefer a full-sync backfill only when the feature needs library-wide consistency and there is no safe incremental path.

## Sync And GitHub API Rules

- Keep incremental sync and rescan aligned with authenticated REST `GET /user/starred`; that endpoint matches the current cursor and tombstone model.
- Keep full sync, incremental sync, and rescan aligned with authenticated REST `GET /user/starred` whenever the required metadata already exists there.
- `archived` must come from GitHub metadata (`repo.archived` or GraphQL `isArchived`) and be stored locally; never guess it from UI state.
- Repository creation time should come from GitHub repo metadata (`created_at` / `createdAt`), not from releases, tags, or first-star heuristics.
- Preserve tombstone semantics. By default the product operates on currently starred repos, not historical unstarred rows.

## GitHub Docs To Trust

- REST starring endpoints: `https://docs.github.com/v3/activity/starring`
- REST repositories endpoints: `https://docs.github.com/rest/repos/repos`

## Privacy And External Context


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [izumi0uu/better-github-stars-manager](https://github.com/izumi0uu/better-github-stars-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
