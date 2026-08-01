---
trigger: always_on
description: This repository stores curated OpenUPM package data and the validation wrapper used by CI.
---

# Repository Guidelines

## Project Structure & Module Organization

This repository stores curated OpenUPM package data and the validation wrapper used by CI.

- `data/packages/*.yml`: one package manifest per Unity package, named after the package id, for example `com.example.tool.yml`.
- `data/*.yml`: shared metadata lists such as sponsors, backers, built-in packages, and blocked scopes.
- `test/data-packages.js`: Node test that runs the shared validator from `openupm-next`.
- `package.json` and `package-lock.json`: npm metadata and pinned test dependencies.

## Build, Test, and Development Commands

- `npm install`: install dependencies from `package-lock.json`.
- `npm test`: run the full repository test suite. Currently this aliases `npm run test:data`.
- `npm run test:data`: validate `data/` with the built validator from `openupm-next`.

The data test expects a built `openupm-next` checkout next to this repository at `../openupm-next`, or set `OPENUPM_NEXT_PATH=/path/to/openupm-next`. Build `openupm-next` before running validation locally.
When validating against an `openupm-next` feature worktree, set
`OPENUPM_NEXT_PATH` to that worktree and ensure its
`packages/@openupm/local-data/build/cli/validate-data.js` has been rebuilt.

When searching, remember `rg` skips hidden paths by default. Use `rg --hidden ...` when hidden project directories need to be included.

When staging the `openupm-next/apps/docs` dev server for browser testing, bind to `0.0.0.0` so LAN users can visit it.

## Coding Style & Naming Conventions

Use JavaScript CommonJS style for tests and scripts, matching the existing `require(...)` pattern. Use two-space indentation in YAML package files and keep package keys in the established order where practical.

Package manifest filenames must match the package `name` field, for example `data/packages/com.vendor.package.yml`. Prefer lowercase Unity package ids and existing topic names.

## Testing Guidelines

Tests use Node's built-in `node:test` module and `node:assert/strict`. Add focused tests under `test/` when changing validation behavior. For data-only changes, run `npm test` and ensure the shared validator reports no schema, tracking, license, or metadata errors.

## Commit & Pull Request Guidelines

Use `chore(data): ...` for package CRUD and metadata-only changes, for example `chore(data): add com.vendor.package` or `chore(data): delist com.vendor.package`. Because package submission PRs are often auto-merged, history may also contain generated subjects such as `Create com.vendor.package.yml` or `Rename package from old.id to new.id`; do not treat those as the preferred manual style.

Pull requests should describe the package or metadata change, link related issues when available, and include validation results. For package additions or renames, mention the package id, repository URL, tracking mode, license, and any localization fields touched. Screenshots are only useful when changing images or display metadata.
Do not include machine-local absolute paths in PR titles, bodies, comments, or
issues. Replace local worktree paths with placeholders such as
`<openupm-next-worktree>` or use repo-relative paths.
Before opening a PR from a worktree, check `git log --oneline origin/master..HEAD`
and the changed-file list. The canonical `master` branch may contain local
operator commits, so feature branches should be rebased onto `origin/master`
when those commits are unrelated to the data change.

Before creating GitHub issues or opening PRs that manipulate package data, read `.github/ISSUE_TEMPLATE/`. The templates document expected workflows for package replacement, unpublish requests, bug reports, feature requests, and abuse reports.

## Security & Configuration Tips

Treat tokens and deploy settings as sensitive. Do not introduce secrets into tracked files; prefer environment variables or ignored local configuration.

## Pull Request Delivery Workflow

Deliver repository changes through pull requests by default, regardless of
size. Do not make changes directly in the main checkout unless the user
explicitly approves an exception. Direct commits to `main` or the default
branch should be limited to explicit user-approved exceptions.

Follow this delivery sequence:

1. Create a dedicated topic branch. Use a separate worktree when repository
   guidance requires one or when isolation is useful.
2. Make the requested change and run relevant validation.
3. Update plan progress when working from a saved plan.
4. Run the review gate, fix valid findings, revalidate, and repeat the review
   until it passes.
5. Close the plan when appropriate, then commit and push the reviewed change.
6. Create or update the GitHub pull request with a brief summary and the
   validation commands that were run.
7. Verify required checks and merge when there is no blocking reason.
8. Monitor any explicitly authorized deployment when applicable, then remove
   the clean merged worktree and delete its merged local and remote topic
   branches. Ordinary remote deletion is authorized after confirming that the
   exact pull request is merged and the remote ref matches its recorded head.
   After a squash merge, `git branch -D` is authorized only for the local topic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openupm/openupm](https://github.com/openupm/openupm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
