---
trigger: always_on
description: SeedSync is a Docker-based tool that syncs files from a remote seedbox to a local machine using LFTP.
---

# SeedSync - Claude Code Instructions

## Project Overview

SeedSync is a Docker-based tool that syncs files from a remote seedbox to a local machine using LFTP.

- **Frontend**: Angular 21 (Bootstrap 5.3, Font Awesome 7, Vitest)
- **Backend**: Python 3.13 with Bottle
- **Container**: Multi-arch Docker image (amd64, arm64)
- **Registry**: ghcr.io/nitrobass24/seedsync

## Repository Structure

```
src/
├── angular/          # Angular 21 frontend
├── python/           # Python backend
├── docker/           # Docker build files
└── e2e-playwright/   # Playwright end-to-end tests
website/              # Docusaurus documentation site
```

## Branches

- **master** - Stable release branch
- **develop** - Integration branch for all new work

## Git Workflow

All work MUST follow this branching discipline:

1. **Always start from `develop`**: Before writing any code, checkout `develop` and pull latest:
   ```bash
   git checkout develop && git pull origin develop
   ```
2. **Create a feature branch**: Every task (feature, bugfix, refactor) gets its own branch off `develop`:
   ```bash
   git checkout -b feat/short-description   # or fix/short-description
   ```
3. **Commit only to the feature branch**: Never commit directly to `develop` or `master`.
4. **One concern per branch**: Do not mix unrelated changes into the same branch. If you discover a separate issue while working, finish or stash your current work, then create a new branch for the other issue.
5. **Open a PR to `develop`**: When the work is complete, push the feature branch and open a PR targeting `develop`. Include a summary and test plan. Releases are the only path that PRs to `master` — see the Release Process below.
6. **Do not carry dirty working-tree changes across branches**: Before switching branches, either commit or stash. Never rely on uncommitted edits surviving a `git checkout`.

## Release Process

When merging a PR or completing significant work, follow this release process:

### 1. Update CHANGELOG.md

Add a new version entry at the top with:
- Version number following semver (major.minor.patch)
- Date in YYYY-MM-DD format
- Sections: Changed, Added, Fixed, Removed, Security (as applicable)

Example:
```markdown
## [0.10.0] - 2026-01-27

### Changed
- **Feature name** - Description of change

### Fixed
- **Bug name** - Description of fix
```

### 2. Update MODERNIZATION_PLAN.md

If the change relates to modernization tasks:
- Update task status (🔄 IN PROGRESS → ✅)
- Update version references
- Update architecture diagram if needed

### 3. Update package.json Version

Update the version in `src/angular/package.json` to match the release version. This is displayed on the About page.

### 4. Create Release

Releases use a `release/vX.Y.Z` branch off `develop`, a PR into `master`, and a tag on the resulting merge commit. Never commit the release directly to `develop` or `master`.

```bash
# Branch from the latest develop
git checkout develop && git pull origin develop
git checkout -b release/vX.Y.Z

# Stage the release commit (CHANGELOG / MODERNIZATION_PLAN / package.json)
git add CHANGELOG.md MODERNIZATION_PLAN.md src/angular/package.json
git commit -m "Release vX.Y.Z - Brief description"
git push -u origin release/vX.Y.Z

# Open a PR targeting master
gh pr create --base master --head release/vX.Y.Z \
  --title "Release vX.Y.Z - Brief description" \
  --body "Sync develop → master for vX.Y.Z. See CHANGELOG.md for details."

# After the PR is merged into master, fetch and tag the merge commit
git checkout master && git pull origin master
git tag vX.Y.Z
git push origin vX.Y.Z
```

The CI workflow will automatically:
- Build multi-arch Docker images
- Push to ghcr.io
- Create GitHub release with auto-generated notes

### 5. Update GitHub Release Notes

After CI creates the release, update the release notes with detailed changelog:

```bash
gh release edit vX.Y.Z --repo nitrobass24/seedsync --notes "$(cat <<'EOF'
## What's Changed

### Fixed
- **Bug name** - Description of fix (#issue)

### Changed
- **Feature name** - Description of change

### Added
- **New feature** - Description

## Docker Pull

```bash
docker pull ghcr.io/nitrobass24/seedsync:X.Y.Z
```

**Full Changelog**: https://github.com/nitrobass24/seedsync/compare/vPREV...vX.Y.Z
EOF
)"
```

Format should match CHANGELOG.md entries with:
- Section headers: Fixed, Changed, Added, Removed, Security
- Bold feature/bug names
- Issue references where applicable
- Always include a "Docker Pull" section with the `docker pull` command for the release version

### 6. Verify Release

- Check GitHub Actions completed successfully
- Verify image is available: `docker pull ghcr.io/nitrobass24/seedsync:X.Y.Z`
- Verify GitHub release notes are formatted correctly

## Version Numbering

- **Major** (X.0.0): Breaking changes, major rewrites
- **Minor** (0.X.0): New features, significant improvements (e.g., Angular upgrade)
- **Patch** (0.0.X): Bug fixes, minor updates

## Key Files

| File | Purpose |
|------|---------|
| `CHANGELOG.md` | Release history and notes |
| `MODERNIZATION_PLAN.md` | Project modernization status |
| `src/docker/build/docker-image/Dockerfile` | Multi-stage Docker build |
| `.github/workflows/ci.yml` | CI/CD pipeline |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nitrobass24/seedsync](https://github.com/nitrobass24/seedsync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
