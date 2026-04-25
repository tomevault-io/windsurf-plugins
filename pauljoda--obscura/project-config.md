---
trigger: always_on
description: - **You MUST ALWAYS commit after every set of changes.** Do not wait for the user to ask — commit immediately when a logical unit of work is complete.
---

# Obscura Repository Contract

## Commit & Changelog Policy

- **You MUST ALWAYS commit after every set of changes.** Do not wait for the user to ask — commit immediately when a logical unit of work is complete.
- Every meaningful implementation iteration must end in a git commit.
- Use small, reviewable commits with intentional scopes.
- **With every commit you MUST also** add an entry under `## [Unreleased]` in `CHANGELOG.md`, grouped by the Keep a Changelog sections (`Added` / `Changed` / `Fixed` / `Removed` / `Docs`).
- **Do NOT bump `package.json` versions on regular commits.** The version only changes when a release is cut by the Release workflow. Between releases, the root `package.json` carries a pre-release marker such as `0.14.0-dev`, and all workspace packages mirror it.
- Follow [Keep a Changelog](https://keepachangelog.com/) and [Semantic Versioning](https://semver.org/).
- Suggested commit style:
  - `chore: bootstrap workspace`
  - `docs: define repo contract`
  - `feat(web): add media library shell`
  - `feat(api): add health and jobs routes`
  - `fix(worker): stabilize queue startup`
- Do not batch unrelated changes into a single commit.
- `chore(release): …` commits are reserved for the Release workflow. Never create them by hand.

### Versioning

- The root `package.json` version is the single source of truth. All workspace `package.json` files must match it exactly.
- Between releases, the version is `X.Y.Z-dev` (for example `0.14.0-dev`), meaning "the next release will be at least `X.Y.Z`". Dev Docker images are tagged `ghcr.io/pauljoda/obscura:dev` and `…:<version>-<short-sha>`.
- Release versions are plain `X.Y.Z` with no suffix. They always have a matching `## [X.Y.Z] - YYYY-MM-DD` heading in `CHANGELOG.md`.
- Git release tags are `vX.Y.Z`.
- The Dockerfile runs `pnpm release:check` on every build, and `pnpm release:check --release` on release builds (via the `RELEASE_STRICT=1` build arg), which enforces that a versioned CHANGELOG heading matches `package.json`.
- **MAJOR**: Breaking API changes, DB schema changes requiring manual migration, config format changes.
- **MINOR**: New features, new API endpoints, new UI views.
- **PATCH**: Bug fixes, UI tweaks, dependency updates, docs.

### How to publish a release

Releases are cut server-side by the GitHub Actions `Release` workflow. Do not hand-edit versions, hand-edit CHANGELOG release headings, or push release tags manually.

1. Make sure `main` is green, all the changes you want in the release are merged, and `## [Unreleased]` in `CHANGELOG.md` is populated with human-readable entries — these become the GitHub Release body verbatim.
2. Open the repository on GitHub → **Actions** → **Release** → **Run workflow**.
3. Pick the bump level (`patch` / `minor` / `major`) or paste an explicit `X.Y.Z` version (overrides the bump).
4. The workflow will:
   1. Run `scripts/release/cut.mjs --phase release` on main, which bumps every `package.json` to `X.Y.Z`, converts `## [Unreleased]` into `## [X.Y.Z] - YYYY-MM-DD` (leaving a fresh empty `## [Unreleased]` above it), and writes `RELEASE_NOTES.md`.
   2. Create commit `chore(release): vX.Y.Z` and tag `vX.Y.Z`.
   3. Run `scripts/release/cut.mjs --phase post`, which bumps every `package.json` to the next dev marker (`X.Y.(Z+1)-dev`) and commits `chore(release): begin vX.Y.(Z+1)-dev cycle`.
   4. Push both commits to `main` and push the tag.
   5. Check out the release tag and build the unified Docker image with `RELEASE_STRICT=1`, pushing it to GHCR as `latest`, `X.Y.Z`, `X.Y`, and `X`.
   6. Create a GitHub Release for `vX.Y.Z` with `RELEASE_NOTES.md` as the body.
5. The `Build Dev Image` workflow is wired to skip any commit whose message starts with `chore(release):`, so the release commit and post-release dev-bump commit do not double-trigger a dev build.
6. If the release workflow fails partway through, the tag may already exist. Delete the tag (`git push --delete origin vX.Y.Z`) before retrying. Only do this when you're certain the release image and GitHub Release have not been published.

### Release notes discipline

- **Every release section (including `## [Unreleased]`) MUST begin with a `### What's New` block** before the standard Keep a Changelog groups. This is a plain-language summary of user-visible highlights — written so a non-technical user can scan it and know what changed. Keep each bullet to 1–2 sentences. Only include features, major fixes, or behavioral changes a user would notice; skip internal refactors, code cleanup, and dependency bumps.
- When adding a changelog entry during development: if the change is a new feature, a significant UI change, or a notable bug fix, **also add or update a bullet in `### What's New`**. Minor internal changes (refactors, code splits, dev tooling) only go in the standard sections below.
- The standard Keep a Changelog sections (`### Added`, `### Changed`, `### Fixed`, `### Removed`, `### Docs`) follow after `### What's New` and contain the full detailed entries as before.
- Entries should be written for users of the app, not for the diff reader. Explain user-visible behavior and why it changed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pauljoda/Obscura](https://github.com/pauljoda/Obscura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
